---
layout: post
title: "How to setup PostgreSQL on Arch Linux"
author: Job Hernandez
tags: sql
---
Arch Linux is great; you should check it out. This `how-to` exists mainly as an external memory for me. I hope this could be useful for someone else.

Suppose you are trying to set up PostgreSQL for a Django project you are working on. Setting up PostgreSQL on Arch linux can be a little tricky.

To set up follow the following:

1. Upgrade your system:

```
$ sudo pacman -Syu
```

2. Install PostgreSQL:

```
$ sudo pacman -S postgresql
```

3. Initialize postgresql database:

```
$ sudo -u postgres initdb --locale en_US.UTF-8 -D /var/lib/postgres/data
```

4. Start postgres service with `sytemctl`:

```
$ sudo systemctl start postgresql
```

5. Enable the postgresql service:

```
$ sudo systemctl enable postgresql
```

6. Type the following:

```
$ su - postgres
```

and if you get the error *Authentication failure* then type:

```
$ sudo passwd postgres
```

and type your password.

7. Type the following again:

```
$ su - postgres
```

and finally:

```
$ psql
```
