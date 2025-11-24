---
title: J'utilise les attributs dès que je peux !
parent: SSH
grand_parent: Astuces
nav_order: 1
nav_exclude: true
---

#### Infos :
jump : 192.168.10.55
db-docker-01 : 192.168.10.56
db-docker-02 : 192.168.10.57


#### Machine client (sur laquelle on lance le script bash/python...) :
/etc/hosts
192.168.10.55  jump

#### Machine jump qui servira de bastion :
/etc/hosts
192.168.10.56  db-docker-01
192.168.10.57  db-docker-02


#### Mettre en place un système de clés + agent :
ssh-keygen -t ed25519 -f ~/atelier/oremis/id_ed25519_jump -C "vrm@jump"
ssh-copy-id -i ~/atelier/oremis/id_ed25519_jump.pub vrm@jump

ssh-keygen -t ed25519 -f ~/atelier/oremis/id_ed25519_db_source -C "lotfi@db-docker-01"
ssh-copy-id -i ~/atelier/oremis/id_ed25519_db_source.pub -o "ProxyJump vrm@jump" lotfi@db-docker-01

ssh-keygen -t ed25519 -f ~/atelier/oremis/id_ed25519_db_target -C "lotfi@db-docker-02"
ssh-copy-id -i ~/atelier/oremis/id_ed25519_db_target.pub -o "ProxyJump vrm@jump" lotfi@db-docker-02


Modifier ~/.ssh/config pour automatiser l'utilisation des clés et du jump host :
# 1. Configuration de l'hôte Jump (Bastion)
Host jump
    HostName 192.168.10.55
    User vrm
    IdentityFile ~/atelier/oremis/id_ed25519_jump

# 2. Configuration des hôtes Docker DB
Host db-docker-01
    HostName 192.168.10.56
    User lotfi
    ProxyJump jump  
    IdentityFile ~/atelier/oremis/id_ed25519_db_source

Host db-docker-02
    HostName 192.168.10.57
    User lotfi
    ProxyJump jump
    IdentityFile ~/atelier/oremis/id_ed25519_db_target

#### Ajouter les clés à l'agent ssh :
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/oremis/id_ed25519

#### Script bash pour lancer deux conteneurs mysql sur deux machines distantes via jump host bastion :

```bash
#!/bin/bash
ssh lotfi@db-docker-01 \
    'docker volume create mysql-db1-data && \
     docker run -d --name mysql-db1 --restart unless-stopped \
         -v mysql-db1-data:/var/lib/mysql \
         -e MYSQL_ROOT_PASSWORD=rootpassword \
         -e MYSQL_DATABASE=mydatabase \
         -p 3306:3306 mysql:latest'

ssh lotfi@db-docker-02 \
    'docker volume create mysql-db2-data && \
     docker run -d --name mysql-db2 --restart unless-stopped \
         -v mysql-db2-data:/var/lib/mysql \
         -e MYSQL_ROOT_PASSWORD=rootpassword \
         -e MYSQL_DATABASE=mydatabase \
         -p 3306:3306 mysql:latest'
```

#### Créer une base de donnée et un utilisateur sur les deux serveurs mysql distants :

ssh lotfi@db-docker-01 'docker exec mysql-db1 mysql -u root -prootpassword -e "CREATE USER 'myuser'@'%' IDENTIFIED BY 'mypassword'; GRANT ALL PRIVILEGES ON mydatabase.* TO 'myuser'@'%'; FLUSH PRIVILEGES;"'

ssh lotfi@db-docker-02 'docker exec mysql-db2 mysql -u root -prootpassword -e "CREATE USER 'myuser'@'%' IDENTIFIED BY 'mypassword'; GRANT ALL PRIVILEGES ON mydatabase.* TO 'myuser'@'%'; FLUSH PRIVILEGES;"'

```bash
mysql -u root -prootpassword -e 
CREATE USER 'myuser'@'%' IDENTIFIED BY 'mypassword'; 
GRANT ALL PRIVILEGES ON mydatabase.* TO 'myuser'@'%'; 
FLUSH PRIVILEGES;
```

USE mydatabase;

## Remplir un peu la base de donnée pour une démonstration

```sql
CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS articles (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    content TEXT,
    author_id INT,
    FOREIGN KEY (author_id) REFERENCES users(id)
);

INSERT INTO users (username, email) VALUES
('admin_user', 'admin@domaine.com'),
('testeur_dev', 'dev@test.com'),
('client_regulier', 'client@test.com');

INSERT INTO articles (title, content, author_id) VALUES
('Premier Article de Test', 'Ceci est le contenu du tout premier article.', 1),
('Billet sur les Conteneurs', 'Analyse rapide de l''utilisation des conteneurs Docker pour les bases de données.', 2),
('Procédure de Dump Réussie', 'Test de la procédure de sauvegarde et de restauration des données.', 1);
```

SHOW DATABASES;

SHOW TABLES;

DESCRIBE users;
-- OU
DESC users;

SELECT * FROM users;



ssh -J vrm@jump lotfi@db-docker-02 
mysql -u root -prootpassword -e \"CREATE DATABASE mydatabase; 
CREATE USER 'myuser'@'%' IDENTIFIED BY 'mypassword'; 
GRANT ALL PRIVILEGES ON mydatabase.* TO 'myuser'@'%'; 
FLUSH PRIVILEGES;





ssh -J vrm@jump lotfi@db-docker-01 \
    'docker exec mysql-db1 mysqldump \
    --single-transaction \
    --set-gtid-purged=OFF \
    -u root -prootpassword mydatabase > /tmp/initial_dump.sql'


scp -J vrm@jump lotfi@db-docker-01:/tmp/initial_dump.sql ./initial_dump.sql

scp -J vrm@jump ./initial_dump.sql lotfi@db-docker-02:/tmp/initial_dump.sql

ssh -J vrm@jump lotfi@db-docker-02 \
    'docker exec -i mysql-db2 mysql -u root -prootpassword mydatabase < /tmp/initial_dump.sql'

ssh -J vrm@jump lotfi@db-docker-02 \
    'docker exec -i mysql-db2 mysql \
    -u root -prootpassword mydatabase < /tmp/initial_dump.sql'

cat /etc/passwd | grep lvolet
usermod -s /bin/bash lvolet
cat /etc/passwd | grep lvolet


mkdir /home/lvolet/.ssh
touch /home/lvolet/.ssh/authorized_keys
chmod 750 -R /home/lvolet/.ssh
chmod 600 /home/lvolet/.ssh/authorized_keys
chown -R lvolet:lvolet /home/lvolet/.ssh

