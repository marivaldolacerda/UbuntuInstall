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

