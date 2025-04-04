## Création de la Base de Données

```sql
CREATE DATABASE IF NOT EXISTS location_ski;
USE location_ski;
```

---
## Création des Tables

```sql
CREATE TABLE station (
    id INT NOT NULL AUTO_INCREMENT,
    nom VARCHAR(50) NOT NULL,
    localisation VARCHAR(100) NOT NULL,
    CONSTRAINT pk_station PRIMARY KEY (id)
) ENGINE=INNODB;

CREATE TABLE client (
    id INT NOT NULL AUTO_INCREMENT,
    nom VARCHAR(50) NOT NULL,
    prenom VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    CONSTRAINT pk_client PRIMARY KEY (id)
) ENGINE=INNODB;

CREATE TABLE equipement (
    id INT NOT NULL AUTO_INCREMENT,
    type VARCHAR(50) NOT NULL,
    marque VARCHAR(50) NOT NULL,
    prix_location FLOAT NOT NULL,
    station_id INT NOT NULL,
    CONSTRAINT pk_equipement PRIMARY KEY (id),
    CONSTRAINT fk_equipement_station FOREIGN KEY (station_id) REFERENCES station(id) ON DELETE CASCADE
) ENGINE=INNODB;

CREATE TABLE reservation (
    id INT NOT NULL AUTO_INCREMENT,
    client_id INT NOT NULL,
    equipement_id INT NOT NULL,
    date_debut DATE NOT NULL,
    date_fin DATE NOT NULL,
    montant_total FLOAT NOT NULL,
    CONSTRAINT pk_reservation PRIMARY KEY (id),
    CONSTRAINT fk_reservation_client FOREIGN KEY (client_id) REFERENCES client(id) ON DELETE CASCADE,
    CONSTRAINT fk_reservation_equipement FOREIGN KEY (equipement_id) REFERENCES equipement(id) ON DELETE CASCADE
) ENGINE=INNODB;
```

---
## Insertion des Données

```sql
INSERT INTO station (nom, localisation) VALUES
('Alpes Ski Resort', 'Chamonix'),
('Mont Blanc Ski', 'Courchevel'),
('Snow Paradise', 'Val Thorens');

INSERT INTO client (nom, prenom, email) VALUES
('Dupont', 'Jean', 'jean.dupont@email.com'),
('Martin', 'Sophie', 'sophie.martin@email.com'),
('Durand', 'Paul', 'paul.durand@email.com');

INSERT INTO equipement (type, marque, prix_location, station_id) VALUES
('Ski', 'Rossignol', 25.0, 1),
('Snowboard', 'Burton', 30.0, 2),
('Ski', 'Salomon', 28.0, 3);

INSERT INTO reservation (client_id, equipement_id, date_debut, date_fin, montant_total) VALUES
(1, 1, '2025-02-10', '2025-02-15', 125.0),
(2, 2, '2025-02-12', '2025-02-18', 180.0),
(3, 3, '2025-02-14', '2025-02-20', 168.0);
```

---
## Requêtes SQL

### 1️⃣ Nom commence par un D

```sql
SELECT * FROM client WHERE nom LIKE 'D%';
```

---

### 2️⃣ Nom et prénom de tous les clients

```sql
SELECT prenom, nom FROM client;
```

---

### 3️⃣ Fiches des clients habitant en Loire-Atlantique (44)

```sql
SELECT noFic, etat, nom, prenom
FROM fiche
JOIN client USING(noCli)
WHERE cpo LIKE '44%';
```

---

### 4️⃣ Détail de la fiche n°1002

```sql
SELECT noFic, nom, prenom, refart, designation, depart, retour, prixJour,
(prixJour * DATEDIFF(COALESCE(retour, CURDATE()), depart)) AS montant
FROM ligne
JOIN fiche USING(noFic)
JOIN client USING(noCli)
JOIN article USING(refart)
WHERE noFic = 1002;
```

---

### 5️⃣ Prix journalier moyen de location par gamme

```sql
SELECT gamme, AVG(prixJour) AS tarif_moyen
FROM article
GROUP BY gamme;
```

---

### 6️⃣ Détail de la fiche n°1002 avec le total

```sql
SELECT noFic, nom, prenom, refart, designation, depart, retour, prixJour,
(prixJour * DATEDIFF(COALESCE(retour, CURDATE()), depart)) AS montant,
SUM(prixJour * DATEDIFF(COALESCE(retour, CURDATE()), depart)) OVER (PARTITION BY noFic) AS total
FROM ligne
JOIN fiche USING(noFic)
JOIN client USING(noCli)
JOIN article USING(refart)
WHERE noFic = 1002;
```

---

### 7️⃣ Grille des tarifs

```sql
SELECT libelle, gamme, niveau, prixJour
FROM article
JOIN categorie USING(codeCat);
```

---

### 8️⃣ Locations de la catégorie SURF

```sql
SELECT refart, designation, COUNT(*) AS nbLocation
FROM ligne
JOIN article USING(refart)
JOIN categorie USING(codeCat)
WHERE libelle = 'Surf'
GROUP BY refart, designation;
```

---

### 9️⃣ Nombre moyen d’articles loués par fiche

```sql
SELECT AVG(nb) AS nb_lignes_moyen_par_fiche
FROM (
  SELECT COUNT(*) AS nb
  FROM ligne
  GROUP BY noFic
) AS sousreq;
```

---

### 🔟 Nombre de fiches pour certaines catégories

```sql
SELECT libelle AS catégorie, COUNT(*) AS nb_location
FROM ligne
JOIN article USING(refart)
JOIN categorie USING(codeCat)
WHERE libelle IN ('Ski alpin', 'Surf', 'Patinette')
GROUP BY libelle;
```

---

### 🔟+1 Montant moyen des fiches

```sql
SELECT AVG(total) AS montant_moyen
FROM (
  SELECT noFic, SUM(prixJour * DATEDIFF(COALESCE(retour, CURDATE()), depart)) AS total
  FROM ligne
  JOIN article USING(refart)
  GROUP BY noFic
) AS sousreq;
```
