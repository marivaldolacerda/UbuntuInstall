# UbuntuInstall Particionamento

## Particionar um HD de 3.6Tb

### Listando todos HDs no servidor com o comado:

sudo lsblk

marivaldo@ubsrv01:~$ sudo lsblk
[sudo] password for marivaldo:
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   1.8T  0 disk
sdb      8:16   0   3.6T  0 disk
sdc      8:32   0 446.1G  0 disk
├─sdc1   8:33   0     1G  0 part
└─sdc2   8:34   0 445.1G  0 part /
sdd      8:48   0 892.2G  0 disk

### Agora vamos criar duas partições no disco sdb sendo a primeira de 20% do disco para container e a sunda de 80% para maquinas virtuais.

Como o disco tem >2 TB, use uma tabela de partições GPT (o MBR não suporta esse tamanho). 
Abra o parted no disco:

``sudo parted /dev/sdb``

marivaldo@ubsrv01:~$ sudo parted /dev/sdb
GNU Parted 3.6
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)    

No prompt do parted, crie a tabela GPT e confirme a destruição de dados:

``(parted) mklabel gpt``
``Warning: The existing disk label on /dev/sdb will be destroyed and all data on this disk will be lost. Do you want to continue?``
``Yes/No? Yes``

Em seguida, crie duas partições primárias ext4. Para dividir com 20% para a primeira e 80% para a segunda:

``(parted) mkpart primary ext4 0% 20%``                                       
``(parted) mkpart primary ext4 20% 100%``

Use print para verificar a tabela de partições criada, então saia do parted:

``(parted) print``

(parted) print                                                            
Model: LSI MegaRAID SAS RMB (scsi)
Disk /dev/sdb: 3998GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  800GB   800GB                primary
 2      800GB   3998GB  3198GB               primary

``(parted) quit``

Agora, as partições /dev/sdb1 e /dev/sdb2 devem existir. Confirme com lsblk novamente ou sudo fdisk -l /dev/sdb:

``sudo lsblk``

marivaldo@ubsrv01:~$ sudo lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   1.8T  0 disk 
sdb      8:16   0   3.6T  0 disk 
├─sdb1   8:17   0 744.7G  0 part 
└─sdb2   8:18   0   2.9T  0 part 
sdc      8:32   0 446.1G  0 disk 
├─sdc1   8:33   0     1G  0 part 
└─sdc2   8:34   0 445.1G  0 part /
sdd      8:48   0 892.2G  0 disk 

Isso indica que /dev/sdb1 e /dev/sdb2 foram criadas corretamente.

## Formatar as partições como ext4

Formate cada partição em ext4 usando o mkfs.ext4

``sudo mkfs.ext4 /dev/sdb1``
``sudo mkfs.ext4 /dev/sdb2``

Você pode adicionar etiquetas caso queira

``sudo e2label /dev/sdb1 containers``
``sudo e2label /dev/sdb2 vms``

## Criar os pontos de montagem

Crie os diretórios onde as partições serão montadas. Como pedido, usaremos /containers e /vms. É essencial que esses diretórios existam antes de montar.
Crie-os assim:

``sudo mkdir /containers``
``sudo mkdir /vms``

Isso cria os pontos de montagem. Você pode escolher outros nomes ou caminhos, mas lembre-se de refletir isso no /etc/fstab.

## Editar o /etc/fstab para montagem automática

Abra o arquivo /etc/fstab com um editor de texto (faça backup primeiro)

``sudo cp /etc/fstab /etc/fstab.bak``
``sudo nano /etc/fstab``

Adicione duas linhas ao final do arquivo, especificando as partições, pontos de montagem e opções. Por exemplo:

``/dev/sdb1   /containers  ext4    defaults  0 2``
``/dev/sdb2   /vms         ext4    defaults  0 2``

## Testar montagem com mount -a

Após editar o fstab, teste a montagem automática sem reiniciar:

``sudo mount -a``
``sudo systemctl daemon-reload``

## Verificar montagem com lsblk

Por fim, verifique se as partições estão montadas nos pontos corretos. Execute:
 
``lsblk -f``

marivaldo@ubsrv01:~$ lsblk -f
NAME   FSTYPE FSVER LABEL      UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda                                                                                
sdb                                                                                
├─sdb1 ext4   1.0   containers 424e9521-dec7-4194-8797-35dfb525305d  694.7G     0% /containers
└─sdb2 ext4   1.0   vms        02b958ce-dbe9-48ab-8de4-eb57272e2f61    2.7T     0% /vms
sdc                                                                                
├─sdc1 vfat   FAT32            5175-C491                                 1G     1% /boot/efi
└─sdc2 ext4   1.0              f31568a8-690b-4254-8d4c-2ab3015b6af8  404.1G     2% /
sdd            

