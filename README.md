# Домашнее задание к занятию 2 «Работа с Playbook»

## Подготовка к выполнению

1. * Необязательно. Изучите, что такое [ClickHouse](https://www.youtube.com/watch?v=fjTNS2zkeBs) и [Vector](https://www.youtube.com/watch?v=CgEhyffisLY).
2. Создайте свой публичный репозиторий на GitHub с произвольным именем или используйте старый.
3. Скачайте [Playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
4. Подготовьте хосты в соответствии с группами из предподготовленного playbook.

```
---
clickhouse:
  hosts:
    clickhouse-01:
      ansible_host: 10.129.0.34
      ansible_user: user
      ansible_ssh_private_key_file: /root/.ssh/id_rsa
```

## Основная часть

1. Подготовьте свой inventory-файл `prod.yml`.
2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает [vector](https://vector.dev). Конфигурация vector должна деплоиться через template файл jinja2. От вас не требуется использовать все возможности шаблонизатора, просто вставьте стандартный конфиг в template файл. Информация по шаблонам по [ссылке](https://www.dmosk.ru/instruktions.php?object=ansible-nginx-install). не забудьте сделать handler на перезапуск vector в случае изменения конфигурации!

[site.yml](https://github.com/stepynin-georgy/hw_ansible_2/blob/main/playbook/site.yml)

3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.

Используемые модули:
* ```ansible.builtin.command```
* ```ansible.builtin.file```
* ```ansible.builtin.get_url```
* ```ansible.builtin.unarchive```
* ```ansible.builtin.copy```
* ```ansible.builtin.template```
* ```ansible.builtin.replace```
* ```ansible.builtin.user```
* * ```ansible.builtin.service```

4. Tasks должны: скачать дистрибутив нужной версии, выполнить распаковку в выбранную директорию, установить vector.
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.

Призапуске `ansible-lint site.yml` было 13 ошибок, одно предупреждение.

![изображение](https://github.com/stepynin-georgy/hw_ansible_2/blob/main/img/Screenshot_10.png)

После исправления ошибок запустил еще раз:

![изображение](https://github.com/stepynin-georgy/hw_ansible_2/blob/main/img/Screenshot_12.png)

6. Попробуйте запустить playbook на этом окружении с флагом `--check`.

![изображение](https://github.com/stepynin-georgy/hw_ansible_2/blob/main/img/Screenshot_14.png)

Проверка не проходит после дальше установки Clickhouse, из-за того что они не щагружены на систему.

7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.

После запуска playbook'a подключаемся к целевой системе и проверям службы vector и clickhouse:

![изображение](https://github.com/stepynin-georgy/hw_ansible_2/blob/main/img/Screenshot_15.png)

8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.

```
root@ans-host:/opt/ans_hw2/08-ansible-02-playbook/playbook# ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Clickhouse] **********************************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse distrib] ******************************************************************************************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "gid": 1000, "group": "user", "item": "clickhouse-common-static", "mode": "0644", "msg": "Request failed", "owner": "user", "response": "HTTP Error 404: Not Found", "secontext": "unconfined_u:object_r:user_home_t:s0", "size": 246310036, "state": "file", "status_code": 404, "uid": 1000, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] ******************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Install clickhouse packages] *************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Flush handlers] **************************************************************************************************************************************************************************************

TASK [Wait for clickhouse-server to become available] ******************************************************************************************************************************************************
Pausing for 15 seconds (output is hidden)
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [clickhouse-01]

TASK [Create database] *************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Create vector work directory] ************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Get Vector distrib] **********************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Unzip Vector archive] ********************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Install Vector binary] *******************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Check Vector installation] ***************************************************************************************************************************************************************************
changed: [clickhouse-01]

TASK [Create Vector config vector.toml] ********************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Create vector.service daemon] ************************************************************************************************************************************************************************
changed: [clickhouse-01]

TASK [Modify vector.service file] **************************************************************************************************************************************************************************
--- before: /lib/systemd/system/vector.service
+++ after: /lib/systemd/system/vector.service
@@ -8,7 +8,7 @@
 User=vector
 Group=vector
 ExecStartPre=/usr/bin/vector validate
-ExecStart=/usr/bin/vector
+ExecStart=/usr/bin/vector --config /etc/vector/vector.toml
 ExecReload=/usr/bin/vector validate
 ExecReload=/bin/kill -HUP $MAINPID
 Restart=no

changed: [clickhouse-01]

TASK [Create user vector] **********************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Create Vector data_dir] ******************************************************************************************************************************************************************************
ok: [clickhouse-01]

RUNNING HANDLER [Start Vector service] *********************************************************************************************************************************************************************
changed: [clickhouse-01]

PLAY RECAP *************************************************************************************************************************************************************************************************
clickhouse-01              : ok=16   changed=4    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0   
```

9. Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги. Пример качественной документации ansible playbook по [ссылке](https://github.com/opensearch-project/ansible-playbook). Так же приложите скриншоты выполнения заданий №5-8
10. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-02-playbook` на фиксирующий коммит, в ответ предоставьте ссылку на него.

---

### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.

---
