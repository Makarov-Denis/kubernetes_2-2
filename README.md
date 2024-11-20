# Домашнее задание к занятию "`Хранение в K8s. Часть 2`" - `Макаров Денис`

---

### Задание 1

**Что нужно сделать**

Создать Deployment приложения, использующего локальный PV, созданный вручную.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.
3. Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории. 
4. Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.
5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV.  Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.
5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

Создадим новый namespace для задания:

![ns](https://github.com/user-attachments/assets/71f0c48a-8664-4d15-9fc8-831a92776e71)

Созданный Deployment представлен на скриншоте ниже

![deployment busybox](https://github.com/user-attachments/assets/b6b8ccee-a48c-425d-a0a3-425e0cc55b75)


![get pods](https://github.com/user-attachments/assets/4c456056-609e-4502-b7f7-4528ad4198e2)

Статус пода в состоянии Pending. Ввиду того, что не готов PVC

Создаем PV, PVC и запускаем их командой apply. Выполнение представлено на скриншотах ниже.

![pv-vol](https://github.com/user-attachments/assets/94207c06-7b02-4c04-9dfe-80a5fd01e939)

![pvc-vol](https://github.com/user-attachments/assets/8c7c4d34-0f6c-4283-b55a-11fb68a80947)

![apply pv_vol](https://github.com/user-attachments/assets/17e48817-4829-40c6-9d74-26796f093291)

![apply pvc_vol](https://github.com/user-attachments/assets/2d34b60c-ac60-40f8-a39c-6216f3478919)

Проверяем файл на ноде:

![date log_cluster](https://github.com/user-attachments/assets/ae2b596e-1b70-4ff6-929c-81618f27774e)

Удаляем deployment:

![delete](https://github.com/user-attachments/assets/9afc4100-c424-448c-80ea-c4df7f79bb78)

pv и pvc остались:

![pv_pvc get](https://github.com/user-attachments/assets/2d4127c3-6970-4a5a-8dce-ad8bc06f83e6)

Проверяем файлы на ноде:
![proverka cluster](https://github.com/user-attachments/assets/19e1815b-0954-4c98-8fd6-a56c8f19c493)

![cluster pv](https://github.com/user-attachments/assets/23f9afef-3dea-4efb-a7e5-068da107f126)

Файлы остались на ноде. Во-первых, не были удалены ```pv и pvc```, во-вторых, при конфигурировании ```pv``` использовался режим ```ReclaimPolicy: Retain```. Retain - после удаления PV ресурсы из внешних провайдеров автоматически не удаляются. После удаления ```pv``` файлы так же останутся:

![cluster pv](https://github.com/user-attachments/assets/220ac7bf-afd1-48dc-9270-b4fb9d4d4672)

![dannye](https://github.com/user-attachments/assets/7ffe29fe-d1f0-4310-8e11-b64c820a240a)

------

### Задание 2

**Что нужно сделать**

Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.

1. Включить и настроить NFS-сервер на MicroK8S.
2. Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.
3. Продемонстрировать возможность чтения и записи файла изнутри пода. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

Установим и настроим NFS-сервер:

![nfs](https://github.com/user-attachments/assets/fee67aba-26af-4d0d-8cbf-41e30908fcf9)

Созданный Deployment представлен на скриншоте ниже:

![deployment_nfs](https://github.com/user-attachments/assets/7eec6e52-5566-419b-b66f-77860a47a954)

![pvc_nfs](https://github.com/user-attachments/assets/e91c5cbb-6452-49b7-a94e-c5eed36b2d66)

![apply pv_nfs](https://github.com/user-attachments/assets/78c87274-80cf-4894-b050-4b66470ece12)

StorageClass:

![image](https://github.com/user-attachments/assets/166ba887-1310-49b7-a22c-4eb24d32f64e)

![apply sc](https://github.com/user-attachments/assets/6e4e14d3-e09f-4cae-9132-98c7f7859a6b)

![apply pv_nfs](https://github.com/user-attachments/assets/69b52f16-2943-48d9-a764-3ee293ab9daf)
Проверим автоматическое создание PV:
![get get pv_nfs](https://github.com/user-attachments/assets/beb5ee4f-4a32-44d8-a4cc-bc81b6953779)


Проверим возможность чтения и записи файла изнутри пода:
1. Создадим файл на ноде:

![123](https://github.com/user-attachments/assets/338848c5-2773-4882-9b95-133b2515b69a)

2. Проверим чтение и запись файла изнутри пода:

3. Проверим файл на ноде:

![321](https://github.com/user-attachments/assets/8e76fc0d-8f78-483b-ab7d-8bb19c2062ed)

![finish](https://github.com/user-attachments/assets/409e13cf-23c3-4670-a6de-618e6fff8681)


Запись прошла успешно.
