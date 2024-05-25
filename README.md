# blog-information-automatique
Objectif : faire un site d’information avec des articles autogénérés à partir des dépêches AFP pour siphonner du trafic et gagner de l’argent grâce à la publicité…

Travail à effectuer

1) Lister les donner nécessaires
   •  Depêche AFP
        •	id
        •	titre
        •	contenu
        •	date de création
   •  Article
        •	id
        •	titre
        •	contenu
        •	date de création
        •	id_depêche_afp (relation avec la dépêche AFP)
    •  Illustration
        •	id
        •	url
        •	description
        •	id_article (relation avec l'article)
   •  Tag
        •	id
        •	nom
   •  ArticleTag
        •	id_article
        •	id_tag

2) Le diagramme de classes UML
3) créer le diagramme MCD Merise
https://www.mocodo.net/?mcd=eNpljkEOgjAQRfdzijlAF7JlV8uITSohUNcEaY0kCKS0l_BaXsyKxpC4-_Pn5f-fUfl8iCM1_FCm2F7nJoTeMOym0dvRMzStt8ArLYWiFH3vB7v5LlNw3dtwNnKm4dELs_npOVyGfrmtF2iepzi2dwvyxPOYFtwAkFNBFdfEcFfgt4hhkmC2mQYZ1aKS-w8Vg7Y0SKXOtf4PiXptegE0-kTh
   
5) Création du schéma relationnel avec un ORM (SQLAlchemy)

from sqlalchemy import create_engine, Column, Integer, String, ForeignKey, Table
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship, sessionmaker

# Définir la base déclarative
Base = declarative_base()

# Table d'association pour une relation de plusieurs à plusieurs entre Article et Tag
association_article_tag = Table('association_article_tag', Base.metadata,
    Column('article_id', Integer, ForeignKey('article.id')),
    Column('tag_id', Integer, ForeignKey('tag.id'))
)

class DepecheAFP(Base):
    __tablename__ = 'depeche_afp'
    id = Column(Integer, primary_key=True)
    afp_uuid = Column(String, unique=True)
    contenu = Column(String)
    date = Column(String)
    articles = relationship("Article", back_populates="depeche_afp")

class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer, primary_key=True)
    titre = Column(String)
    contenu = Column(String)
    source = Column(String)
    créé_le = Column(String)
    mis_à_jour_le = Column(String)
    publié_le = Column(String)
    depeche_afp_id = Column(Integer, ForeignKey('depeche_afp.id'))
    depeche_afp = relationship("DepecheAFP", back_populates="articles")
    illustrations = relationship("Illustration", back_populates="article")
    tags = relationship("Tag", secondary=association_article_tag, back_populates="articles")

class Illustration(Base):
    __tablename__ = 'illustration'
    id = Column(Integer, primary_key=True)
    url = Column(String)
    description = Column(String)
    article_id = Column(Integer, ForeignKey('article.id'))
    article = relationship("Article", back_populates="illustrations")

class Tag(Base):
    __tablename__ = 'tag'
    id = Column(Integer, primary_key=True)
    nom = Column(String, unique=True)
    articles = relationship("Article", secondary=association_article_tag, back_populates="tags")

# Créer un moteur et une session
engine = create_engine('sqlite:///blog_info_auto.db')
Base.metadata.create_all(engine)
Session = sessionmaker(bind=engine)
session = Session()

# Exemple d'ajout de données
depeche = DepecheAFP(afp_uuid='12345', contenu='Contenu de la dépêche', date='2024-05-24')
article = Article(titre='Titre de l\'article', contenu='Contenu de l\'article', source='AFP', créé_le='2024-05-24', mis_à_jour_le='2024-05-24', publié_le='2024-05-24', depeche_afp=depeche)
tag = Tag(nom='Actualités')
article.tags.append(tag)
illustration = Illustration(url='http://example.com/image.png', description='Une description de l\'image', article=article)

session.add(depeche)
session.add(article)
session.add(tag)
session.add(illustration)
session.commit()
