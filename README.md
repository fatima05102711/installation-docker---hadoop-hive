# 📦 Installation de Hadoop & Hive avec Docker

> *Projet réalisé dans le cadre de la matière **Fouille de Données***
> Par : **HADJA FATOUMATA CAMARA & FODE MOUSSA FOFANA**

---

## 🧰 Prérequis

| Outil         | Version recommandée |
| ------------- | ------------------- |
| Apache Hadoop | 2.7.2               |
| Apache Hive   | 2.3.2               |
| Docker        | Dernière version    |
| Git           | Dernière version    |
| Java          | 1.8                 |
| OS            | Windows 10 + WSL    |

---

## 🔧 Installation Docker & Hadoop/Hive

### 1. Installer Docker

* Télécharger Docker Desktop : [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)

### 2. Vérification de l'installation

```bash
docker --version
```

### 3. Cloner le projet Docker-Hive

```bash
git clone https://github.com/Yanlou/docker-hive
cd docker-hive
```

### 4. Démarrer les services

```bash
docker-compose up -d
```

---

## 📅 Importation des données dans Hive

### 1. Télécharger et extraire le dataset

```bash
wget https://raw.githubusercontent.com/Yanlou/udacity-hadoop-course/master/Datasets/purchases.txt.gz
gunzip purchases.txt.gz
```

### 2. Copier le fichier dans le conteneur Hive

```bash
docker cp ./purchases.txt docker-hive-hive-server-1:/opt/purchases.txt
```

---

## 🐝 Utilisation de Hive via Beeline

### 1. Ouvrir un terminal Hive

```bash
docker exec -it docker-hive-hive-server-1 /bin/bash
```

### 2. Lancer Beeline

```bash
/opt/hive/bin/beeline -u jdbc:hive2://localhost:10000
```

### 3. Créer la table Hive

```sql
CREATE TABLE purchases (
  `date` STRING,
  `time` STRING,
  store STRING,
  product STRING,
  cost DOUBLE,
  payment STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
```

### 4. Charger les données

```sql
LOAD DATA LOCAL INPATH '/opt/purchases.txt' INTO TABLE purchases;
```

---

## 📊 Requêtes HiveQL

```sql
-- 1. Total des achats
SELECT SUM(cost) AS total_cost FROM purchases;

-- 2. Nombre d'achats par magasin
SELECT store, COUNT(*) AS n_achats FROM purchases GROUP BY store;

-- 3. Coût total par produit
SELECT product, SUM(cost) AS total FROM purchases GROUP BY product;

-- 4. Coût total par produit et magasin
SELECT store, product, SUM(cost) AS total FROM purchases GROUP BY store, product;

-- 5. Top 10 des produits les plus vendus
SELECT product, COUNT(*) AS total_sales FROM purchases GROUP BY product ORDER BY total_sales DESC LIMIT 10;

-- 6. Ventes totales par magasin (ordre décroissant)
SELECT store, SUM(cost) AS total_sales FROM purchases GROUP BY store ORDER BY total_sales DESC;

-- 7. Nombre de ventes par jour
SELECT TO_DATE(date) AS sale_day, COUNT(*) AS nb_ventes FROM purchases GROUP BY TO_DATE(date) ORDER BY sale_day;

-- 8. Coût moyen par produit
SELECT product, AVG(cost) AS avg_cost FROM purchases GROUP BY product;

-- 9. Coût total des achats par méthode de paiement
SELECT payment, SUM(cost) AS total_cost FROM purchases GROUP BY payment;

-- 10. Produits les plus vendus le lundi
SELECT product, COUNT(*) AS nb_ventes
FROM purchases
WHERE dayofweek(date) = 2
GROUP BY product
ORDER BY nb_ventes DESC;
```

---

## 🌐 Interface Web Hue (optionnelle)

### Installation rapide

```bash
docker pull gethue/hue:latest
docker run -it -p 8888:8888 gethue/hue:latest bash
```

### Dans le conteneur Hue :

```bash
source build/env/bin/activate
pip install Werkzeug
./build/env/bin/hue runserver_plus 0.0.0.0:8888
```

> 📌 Interface accessible via : [http://localhost:8888](http://localhost:8888)

---

## 🛠️ Commandes Docker utiles

```bash
# Voir les conteneurs actifs
docker ps

# Arrêter un conteneur
docker stop <nom/id>

# Redémarrer un conteneur
docker start <nom/id>

# Logs d’un conteneur
docker logs <nom/id>

# Supprimer un conteneur
docker rm <nom/id>
```

### 🔄 Sauvegarde / Restauration

```bash
# Créer une image à partir d'un conteneur
docker commit <id_conteneur> my-image:latest

# Sauvegarder les données Hive
docker exec hive-server tar czf /tmp/backup.tar.gz /opt/hive/warehouse
docker cp hive-server:/tmp/backup.tar.gz ./backup.tar.gz
```

---

## ✅ Bonnes pratiques

* 📀 **Persistance** : utiliser `volumes` Docker pour ne pas perdre vos données
* 🔐 **Sécurité** : éviter d’exposer les ports sans authentification
* 📊 **Monitoring** : utilisez `docker stats` pour surveiller l'utilisation mémoire et CPU
* 🧠 **Performance** : allouer au moins 4 Go de RAM à Docker pour éviter les erreurs de mémoire
* 📁 **Structure de projet** : créer un dossier `data/`, `logs/`, `scripts/` pour organiser vos fichiers

---

## 🧪 Dépannage

| Problème                     | Solution                                     |
| ---------------------------- | -------------------------------------------- |
| ❌ Beeline ne se connecte pas | Vérifiez que HiveServer2 est lancé           |
| ⚠️ Manque de mémoire Docker  | Augmentez la RAM allouée dans Docker Desktop |
| 🔐 Problèmes de permissions  | Vérifiez les droits sur les fichiers copiés  |
| 📋 Voir les logs Hive        | `docker exec hive-server cat /tmp/hive.log`  |
| 🔎 État des services Hadoop  | `docker exec hive-server jps`                |

---

## 👩‍🏫 Auteurs

* **HADJA FATOUMATA CAMARA**
* **FODE MOUSSA FOFANA**

---

## 📌 Références

* [Yanlou/docker-hive](https://github.com/Yanlou/docker-hive)
* [Udacity Hadoop Course](https://github.com/Yanlou/udacity-hadoop-course)
* [GetHue](https://gethue.com)
