# LEMP-test

## ЗАДАНИЕ
Разработать ansible, запускающий докеризированное веб приложение (nginx, php, mysql) с использованием docker-compose.yml на удалённом сервере.

## ВЫПОЛНЕНИЕ
Для начала создаем вручную ВМ в яндекс облаке, на которой ансибл выполняет установку докера и docker-compose

Затем ансибл копирует из репозитория конфигурационные файлы и docker-compose.yml

В итоге проект получился таким:
https://github.com/AlexanderM33/LEMP-test/tree/main/docker-compose-LEMP


<details close>
<summary>playbook.yml</summary>
<br>
 
 ```
---
- name: My test task - installing LEMP using docker-compose
  hosts: all
  vars:
    server_hostname: www.my-test-task.com
  tasks:
    - name: install packages
      yum: name={{ item }} state=latest update_cache=yes
      with_items:
        - git
        - wget
        - curl

    - name: Add Docker repo
      get_url:
        url: https://download.docker.com/linux/centos/docker-ce.repo
        dest: /etc/yum.repos.d/docker-ce.repo
      become: yes
 
    - name: Install Docker
      package:
        name: docker-ce
        state: latest
      become: yes
 
    - name: Start Docker service
      service:
        name: docker
        state: started
        enabled: yes
      become: yes
 
    - name: Add user to docker group
      user:
        name: user
        groups: docker
        append: yes
      become: yes

    - name: install docker-compose
      shell: sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && sudo chmod +x /usr/local/bin/docker-compose

    - name: Start docker daemon
      service: name=docker state=started enabled=yes

    - name: Deploy site files from Github repository
      git: repo=https://github.com/AlexanderM33/LEMP-test.git dest=/home/user/docker-compose-lemp-stack update=yes force=yes

    - name: deploy Docker Compose stack
      community.docker.docker_compose_v2:
        project_src: /home/user/docker-compose-lemp-stack/docker-compose-LEMP/
        files:
        - docker-compose.yml
 ```
 </details>


<details close>
<summary>docker-compose.yml</summary>
<br>
 
 ```
version: '3.8'

# Services
services:

    # PHP Service
    php:
        build:
            dockerfile: php-dockerfile
        volumes:
            - './php-files:/var/www/html'
        depends_on:
            - mariadb

    # Nginx Service
    nginx:
        image: nginx:latest
        ports:
            - 80:80
        links:
            - 'php'
        volumes:
            - './php-files:/var/www/html'
            - './nginx-conf:/etc/nginx/conf.d'
        depends_on:
            - php

    # MariaDB Service
    mariadb:
        image: mariadb:10.9
        environment:
            MYSQL_ROOT_PASSWORD: your_password
        volumes:
            - mysqldata:/var/lib/mysql

    # phpMyAdmin Service
    phpmyadmin:
        image: phpmyadmin/phpmyadmin:latest
        ports:
            - 8080:80
        environment:
            PMA_HOST: mariadb
        depends_on:
            - mariadb

# Volumes
volumes:

  mysqldata:
 ```
 </details>

<details close>
<summary>Dockerfile</summary>
<br>
 
 ```
FROM php:8.2-fpm

# Installing dependencies for the PHP modules
RUN apt-get update && \
    apt-get install -y zip libzip-dev libpng-dev

# Installing additional PHP modules
RUN docker-php-ext-install mysqli pdo pdo_mysql gd zip
 ```
 </details>

![1-ip-yc](https://github.com/user-attachments/assets/c3736bdd-4f07-4657-b03f-dd064291d9aa)

![docker-ps](https://github.com/user-attachments/assets/85d07510-97a1-4830-9324-041767522382)

![php-admin](https://github.com/user-attachments/assets/d7880fe3-bf52-4258-b4a4-5e298332abeb)

![php-start](https://github.com/user-attachments/assets/977059fa-5f83-46a9-b834-dbb092ee1f3f)
