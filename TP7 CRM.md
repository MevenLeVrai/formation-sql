# Base de Données CRM

## Création de la base de données et des tables

```sql
CREATE DATABASE IF NOT EXISTS crm;

USE CRM;

CREATE TABLE projet(
    id INT NOT NULL AUTO_INCREMENT,
    nom VARCHAR(50) NOT NULL,
    CONSTRAINT kp_projet PRIMARY KEY (id)
) ENGINE=INNODB;

CREATE TABLE client(
    id INT NOT NULL AUTO_INCREMENT,
    nom VARCHAR(50) NOT NULL,
    projet_id INT NOT NULL,
    CONSTRAINT kp_client PRIMARY KEY (id),
    CONSTRAINT fk_client_projet FOREIGN KEY (projet_id) REFERENCES projet(id) ON DELETE CASCADE
) ENGINE=INNODB;

CREATE TABLE devis(
    id INT NOT NULL AUTO_INCREMENT,
    nom VARCHAR(50) NOT NULL,
    info VARCHAR(50) NOT NULL,
    version VARCHAR(50) NOT NULL,
    projet_id INT NOT NULL,
    CONSTRAINT kp_devis PRIMARY KEY (id),
    CONSTRAINT fk_devis_projet FOREIGN KEY (projet_id) REFERENCES projet(id) ON DELETE CASCADE
) ENGINE=INNODB;

CREATE TABLE facture(
    id INT NOT NULL AUTO_INCREMENT,
    devis_id INT NOT NULL,
    nom VARCHAR(50) NOT NULL,
    montant FLOAT NOT NULL,
    date DATE NOT NULL,
    paiement DATE NULL,
    CONSTRAINT kp_facture PRIMARY KEY (id),
    CONSTRAINT fk_facture_devis FOREIGN KEY (devis_id) REFERENCES devis(id) ON DELETE CASCADE
) ENGINE=INNODB;
```

## Insertion de données

```sql
INSERT INTO projet (nom) VALUES
('Creation de site internet'),
('Logiciel CRM'),
('Logiciel de devis'),
('Site internet ecommerce'),
('logiciel ERP'),
('logiciel Gestion de Stock');

INSERT INTO client (nom, projet_id) VALUES
('Mairie de Rennes', 1),
('Neo Soft', 2),
('Sopra', 3),
('Accenture', 4),
('Neo Soft', 5),
('Amazon', 6);

INSERT INTO devis (nom, info, version, projet_id) VALUES
('DEV2100A', 'Site internet partie 1', '1', 1),
('DEV2100B', 'Site internet partie 2', '2', 1),
('DEV2100C', 'Logiciel CRM', '1', 2),
('DEV2100D', 'Logiciel devis', '1', 3),
('DEV2100E', 'Site internet ecommerce', '1', 4),
('DEV2100F', 'logiciel ERP', '1', 5),
('DEV2100G', 'logiciel Gestion de Stock', '1', 6);

INSERT INTO facture (devis_id, nom, montant) VALUES
(1, 'FA001', 1500),
(1, 'FA002', 1500),
(3, 'FA003', 5000),
(4, 'FA004', 3000),
(5, 'FA005', 5000),
(6, 'FA006', 2000);
```

## Requêtes SQL

### 1. Liste des factures avec le nom du client

```sql
SELECT f.id AS facture_id, f.nom AS facture_nom, f.montant, c.nom AS client_nom
FROM facture f
JOIN devis d ON f.devis_id = d.id
JOIN client c ON d.projet_id = c.projet_id;
```

### 2. Nombre de factures par client

```sql
SELECT c.nom AS client,
       COUNT(f.id) AS nb_factures
FROM client c
LEFT JOIN projet p ON c.projet_id = p.id
LEFT JOIN devis d ON p.id = d.projet_id
LEFT JOIN facture f ON d.id = f.devis_id
GROUP BY c.nom
ORDER BY nb_factures DESC;
```

### 3. Chiffre d'affaires par client

```sql
SELECT c.nom AS client,
       SUM(f.montant) AS ca_par_client
FROM client c
LEFT JOIN projet p ON c.projet_id = p.id
LEFT JOIN devis d ON p.id = d.projet_id
LEFT JOIN facture f ON d.id = f.devis_id
GROUP BY c.nom;
```

### 4. Chiffre d'affaires total

```sql
SELECT SUM(montant) AS ca_total FROM facture;
```

### 5. Total des factures non payées

```sql
SELECT SUM(montant) AS total_factures
FROM facture
WHERE paiement IS NULL;
```

### 6. Factures impayées depuis plus de 30 jours

```sql
SELECT nom AS facture,
       DATEDIFF(NOW(), date) AS nb_jour
FROM facture
WHERE paiement IS NULL
AND DATEDIFF(NOW(), date) > 30;
```

### 7. Clients avec factures impayées depuis plus de 30 jours

```sql
SELECT c.nom AS client,
       f.nom AS facture,
       DATEDIFF(NOW(), f.date) AS nb_jour
FROM client c
JOIN projet p ON c.projet_id = p.id
JOIN devis d ON p.id = d.projet_id
JOIN facture f ON d.id = f.devis_id
WHERE f.paiement IS NULL
AND DATEDIFF(NOW(), f.date) > 30;
```

### 8. Calcul des pénalités pour factures impayées

```sql
SELECT c.nom AS client,
       f.nom AS facture,
       DATEDIFF(NOW(), f.date) AS nb_jour,
       DATEDIFF(NOW(), f.date) * 2 AS penalite
FROM client c
JOIN projet p ON c.projet_id = p.id
JOIN devis d ON p.id = d.projet_id
JOIN facture f ON d.id = f.devis_id
WHERE f.paiement IS NULL
AND DATEDIFF(NOW(), f.date) > 30;
```

### 9. Moyenne du chiffre d'affaires par client

```sql
SELECT AVG(ca_par_client) AS moyenne_ca_client
FROM (
    SELECT c.nom AS client,
           SUM(f.montant) AS ca_par_client
    FROM client c
    LEFT JOIN projet p ON c.projet_id = p.id
    LEFT JOIN devis d ON p.id = d.projet_id
    LEFT JOIN facture f ON d.id = f.devis_id
    GROUP BY c.nom
) AS ca_clients;
```

