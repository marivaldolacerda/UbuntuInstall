# Passo a Passo para Instalar o LXD no Ubuntu Server

## Atualize seu sistema

``sudo apt update && sudo apt upgrade -y``

## Instale o Snapd (caso não esteja instalado)

``sudo apt install snapd -y``

## Instale o LXD via Snap (forma recomendada)

``sudo snap install lxd``

## Adicione seu usuário ao grupo lxd

``sudo usermod -aG lxd $USER``

## Depois disso, reinicie a sessão (logout/login) ou:

``newgrp lxd``

## Inicialize o LXD com lxd init

``lxd init``

