# UbuntuInstall Particionamento

## Particionando um HD de 3.6Tb

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