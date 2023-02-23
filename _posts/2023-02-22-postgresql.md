---
layout: post
title: "How to setup PostgreSQL on Arch Linux"
author: Job Hernandez
---

I am new to Arch linux and I needed to setup PostgreSQL for a Django web app I am working on and it took me a while to find how to setup PostgreSQL on my 
Arch Linux machine. But thankfully I was able to glue different sources to make it work.


Anyways, Suppose you are trying to set up PostgreSQL for a Django project you are working on. Setting up PostgreSQL on Arch linux can be a little tricky.

To set up follow the following:

1. Upgrade your system:

```
sudo pacman -Syu
```

2. Install PostgreSQL:

```
sudo pacman -S postgresql
```

3. Create the data directory:

```
sudo mkdir /var/lib/postgres/data
```

4. Set ownership to user `postgres`:

```
chown postgres /var/lib/postgres/data
```

5. As the user `postgres` start the database:

```
sudo -i -u postgres
initdb  -D '/var/lib/postgres/data'
```

6. Start the server

```
sudo systemctl start postgresql

sudo systemctl status postgresql
```

7. Enable postgres

```
sudo systemctl enable postgresql
```

8. Trigger PostgreSQL's shell

```
sudo -u postgres psql
```

And that is all; at this point you can create your database for your given project.
