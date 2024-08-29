**Подготовка к выполнению**

1. Необязательно. Познакомьтесь с LightHouse.

2. Создайте два пустых публичных репозитория в любом своём проекте: vector-role и lighthouse-role.

3. Добавьте публичную часть своего ключа к своему профилю на GitHub.


**Основная часть**

Ваша цель — разбить ваш playbook на отдельные roles.

Задача — сделать roles для ClickHouse, Vector и LightHouse и написать playbook для использования этих ролей.

Ожидаемый результат — существуют три ваших репозитория: два с roles и один с playbook.

*Что нужно сделать*

1. Создайте в старой версии playbook файл requirements.yml и заполните его содержимым:
```
  - src: git@github.com:AlexeySetevoi/ansible-clickhouse.git
    scm: git
    version: "1.13"
    name: clickhouse
```

2. При помощи ansible-galaxy скачайте себе эту роль.


3. Создайте новый каталог с ролью при помощи ansible-galaxy role init vector-role.

4. На основе tasks из старого playbook заполните новую role. Разнесите переменные между vars и default.

5. Перенести нужные шаблоны конфигов в templates.

6. Опишите в README.md обе роли и их параметры. Пример качественной документации ansible role по ссылке.

7. Повторите шаги 3–6 для LightHouse. Помните, что одна роль должна настраивать один продукт.

8. Выложите все roles в репозитории. Проставьте теги, используя семантическую нумерацию. Добавьте roles в requirements.yml в playbook.

9. Переработайте playbook на использование roles. Не забудьте про зависимости LightHouse и возможности совмещения roles с tasks.

10. Выложите playbook в репозиторий.

11. В ответе дайте ссылки на оба репозитория с roles и одну ссылку на репозиторий с playbook.




**Решение**

Для начала по традиции установим 3 машины под управлением Centos так как с Ubuntu не слодилось с первого задания.

Вписываем ip адреса созданных машин в инвентори-фаил

![alt text](https://github.com/mezhibo/Asible-work-role/blob/85a362a6aadb3e295db73868b31809f43bbea921/IMG/1.jpg)


Далее для того чтобы перенести значения из старого плей-файла в роли, создадим иерархию папок и проинициализируем их для каждой роли

![alt text](https://github.com/mezhibo/Asible-work-role/blob/85a362a6aadb3e295db73868b31809f43bbea921/IMG/2.jpg)


Распихаем по частям в нужные директории содержимое плей-файла

По итогу в плей-файле у нас остается содержимое с запуском ролей для Lighthouse, Clickhouse и Vector, а Nginx так и установим без роли через плей

```
---
- name: Install clickhouse
  hosts: clickhouse
  roles:
    - role: clickhouse-role

- name: Install vector
  hosts: vector
  roles:
    - role: vector-role

- name: Install lighthouse
  hosts: lighthouse

  handlers:
    - name: Start nginx service
      become: true
      ansible.builtin.service:
        name: nginx
        state: restarted
  pre_tasks:
    - name: Install epel-release | Install Nginx
      become: true
      yum:
        name: epel-release
        state: present
    - name: Install Nginx | Install Nginx
      become: true
      yum:
        name: nginx
        state: present
      notify: Start nginx service
    - name: Create Nginx config | Install Nginx
      become: true
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        mode: 0644
      notify: Start nginx service

  roles:
    - role: lighthouse-role

  post_tasks:
    - name: Show connect URL lighthouse
      debug:
        msg: "http://{{ ansible_host }}/#http://{{ hostvars['clickhouse-01'].ansible_host }}:8123/?user={{ clickhouse_user }}"

```


Теперь запускаем его, и видим что все тоже самое что было в содержимом одного плея, отработало через роли


![alt text](https://github.com/mezhibo/Asible-work-role/blob/85a362a6aadb3e295db73868b31809f43bbea921/IMG/3.jpg)


![alt text](https://github.com/mezhibo/Asible-work-role/blob/85a362a6aadb3e295db73868b31809f43bbea921/IMG/4.jpg)



**ССЫЛКИ НА РОЛИ**

 - [CLICKHOUSE](https://github.com/mezhibo/mnt-homeworks/tree/2bb9ad07a14e1a0b0333f3b7fe404dfd675c2938/08-ansible-04-role/clickhouse-role)

 - [LIGHTHOUSE](https://github.com/mezhibo/mnt-homeworks/tree/2bb9ad07a14e1a0b0333f3b7fe404dfd675c2938/08-ansible-04-role/lighthouse-role)

 - [VECTOR](https://github.com/mezhibo/mnt-homeworks/tree/2bb9ad07a14e1a0b0333f3b7fe404dfd675c2938/08-ansible-04-role/vector-role)



