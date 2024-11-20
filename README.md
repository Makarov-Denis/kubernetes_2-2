![ns](https://github.com/user-attachments/assets/e8a49971-bdc3-40f9-a085-10ce38aabcff)# Домашнее задание к занятию "`Хранение в K8s. Часть 2`" - `Макаров Денис`

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



Deployment:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netology-app-multitool
  namespace: dz2-2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        volumeMounts:
          - name: my-vol-pvc
            mountPath: /multitool_dir
      volumes:
        - name: my-vol-pvc
          persistentVolumeClaim:
            claimName: my-pvc-nfs
```
```bash
nikulinn@nikulin:~/other/kuber_2-2/scr$ kubectl apply -f deployment_multitool_nfs.yaml 
deployment.apps/netology-app-multitool created

nikulinn@nikulin:~/other/kuber_2-2/scr$ kubectl get pods -n dz2-2
NAME                                      READY   STATUS    RESTARTS   AGE
netology-app-multitool-5bfc76ff89-r89kt   1/1     Running   0          13s
```
StorageClass:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-csi
provisioner: nfs.csi.k8s.io
parameters:
  server: 158.160.40.165
  share: /srv/nfs
reclaimPolicy: Delete
volumeBindingMode: Immediate
mountOptions:
  - hard
  - nfsvers=4.1
```
```bash
nikulinn@nikulin:~/other/kuber_2-2/scr$ kubectl apply -f sc_nfs.yaml 
storageclass.storage.k8s.io/nfs-csi created

nikulinn@nikulin:~/other/kuber_2-2/scr$ kubectl get sc
NAME      PROVISIONER                            RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs       cluster.local/nfs-server-provisioner   Delete          Immediate           true                   24m
nfs-csi   nfs.csi.k8s.io                         Delete          Immediate           false                  20s
```
PVC:
```yaml
# pvc-nfs.yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc-nfs
  namespace: dz2-2
spec:
  storageClassName: nfs-csi
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 2Gi
```
```bash
nikulinn@nikulin:~/other/kuber_2-2/scr$ kubectl apply -f pvc_nfs.yaml 
persistentvolumeclaim/my-pvc-nfs created

nikulinn@nikulin:~/other/kuber_2-2/scr$ kubectl get pvc -n dz2-2
NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
my-pvc-nfs   Bound    pvc-cd550d34-58fa-4a59-a778-617deb65b7a1   2Gi        RWO            nfs-csi        27s

nikulinn@nikulin:~/other/kuber_2-2/scr$ kubectl describe pvc my-pvc-nfs -n dz2-2
Name:          my-pvc-nfs
Namespace:     dz2-2
StorageClass:  nfs-csi
Status:        Bound
Volume:        pvc-cd550d34-58fa-4a59-a778-617deb65b7a1
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: nfs.csi.k8s.io
               volume.kubernetes.io/storage-provisioner: nfs.csi.k8s.io
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      2Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Used By:       <none>
Events:
  Type    Reason                 Age                From                                                             Message
  ----    ------                 ----               ----                                                             -------
  Normal  Provisioning           49s                nfs.csi.k8s.io_netology-01_f632689b-9cc4-4f7b-93d1-896930435435  External provisioner is provisioning volume for claim "dz2-2/my-pvc-nfs"
  Normal  ExternalProvisioning   49s (x2 over 49s)  persistentvolume-controller                                      Waiting for a volume to be created either by the external provisioner 'nfs.csi.k8s.io' or manually by the system administrator. If volume creation is delayed, please verify that the provisioner is running and correctly registered.
  Normal  ProvisioningSucceeded  48s                nfs.csi.k8s.io_netology-01_f632689b-9cc4-4f7b-93d1-896930435435  Successfully provisioned volume pvc-cd550d34-58fa-4a59-a778-617deb65b7a1
```
Проверим автоматическое создание PV:
```bash
nikulinn@nikulin:~/other/kuber_2-2/scr$ kubectl get pv -n dz2-2
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                  STORAGECLASS   REASON   AGE
data-nfs-server-provisioner-0              1Gi        RWO            Retain           Bound    nfs-server-provisioner/data-nfs-server-provisioner-0                           28m
pvc-cd550d34-58fa-4a59-a778-617deb65b7a1   2Gi        RWO            Delete           Bound    dz2-2/my-pvc-nfs                                       nfs-csi                 2m17s
```
Проверим возможность чтения и записи файла изнутри пода:
1. Создадим файл на ноде:
```bash
debian@netology-01:/srv/nfs/pvc-cd550d34-58fa-4a59-a778-617deb65b7a1$ sudo -i

root@netology-01:~# cd /srv/nfs/pvc-cd550d34-58fa-4a59-a778-617deb65b7a1/

root@netology-01:/srv/nfs/pvc-cd550d34-58fa-4a59-a778-617deb65b7a1# echo 123 >> test.txt

root@netology-01:/srv/nfs/pvc-cd550d34-58fa-4a59-a778-617deb65b7a1# cat test.txt 
123
```
2. Проверим чтение и запись файла изнутри пода:
```bash
nikulinn@nikulin:~/other/kuber_2-2/scr$ kubectl exec netology-app-multitool-5bfc76ff89-r89kt -n dz2-2 -it -- bin/bash

netology-app-multitool-5bfc76ff89-r89kt:/# echo 321 >> multitool_dir/test2.txt

netology-app-multitool-5bfc76ff89-r89kt:/# ls -l multitool_dir/
total 8
-rw-r--r--    1 root     root             4 Feb 19 21:37 test.txt
-rw-r--r--    1 nobody   nobody           4 Feb 19 22:02 test2.txt

netology-app-multitool-5bfc76ff89-r89kt:/# cat multitool_dir/test2.txt 
321

```
3. Проверим файл на ноде:
```bash
root@netology-01:~# ls -l /srv/nfs/pvc-cd550d34-58fa-4a59-a778-617deb65b7a1/
total 8
-rw-r--r-- 1 nobody nogroup 4 Feb 19 22:02 test2.txt
-rw-r--r-- 1 root   root    4 Feb 19 21:37 test.txt

root@netology-01:~# cat /srv/nfs/pvc-cd550d34-58fa-4a59-a778-617deb65b7a1/test2.txt 
321
```
Запись прошла успешно.
