# 🛠️ Guía: Cómo Minar ZK con Boundless

## 📋 Requisitos de hardware

| Componente | Recomendación             |
|------------|---------------------------|
| Sistema    | Ubuntu 22.04 LTS   |
| RAM        | 16 GB                 |
| CPU        | 8 núcleos 16 Threads             |
| Disco      | 100 GB SSD     

---

Esta guía corre un nodo minero o prover de boundless usando docker y utiliza un RPC de la Red donde vallamos a trabajar


Puedes seguir esta guía en tu equipo local, o puedes tambien utilizar un VPS (alquilar un servidor virtual) para tener tu nodo/mineroi.

## Redes Disponilbes

### Compatible con:

- Ethereum Sepolia

- Base Sepolia

- Base Mainnet


Para poder minar con Boundless necxesitas tener un RPC de la Red donde vallamos a minar, puedes correr tu ropio nodo y usar tu RPC (ideal) so puedes usar un proveedor de RPC teniendo en cuenta que debes tener un plan pago para evitar errores por limitaciones.

Por EJ. si deseas correr Boundless en Ethereum Sepolia, necesitas un RPC de sepolia quepuedes comprar o correr tu mismo con esta guía de sepolia: https://github.com/wkss1/nodo-sepolia/tree/main

Un RPC no es mas que una llamada que permite interactuar con el nodo, en el caso de Boundless, tener un RPC te permite interactuar directamente y tener acceso a la blockchain para resolver las pruebas y publicar los resultados.

## Cómo funciona boundless

- Publicación de órdenes: Devs suben órdenes con tareas computacionales y recompensas.

- Stake de USDC: Los nodos deben hacer depósito como garantía.

- Puja (mcycle_price): Se compite por probar órdenes.

- Lock-in: El ganador bloquea la orden.

- Generación de prueba (Bento): Se ejecuta con GPU.

- Recompensas o slashing: Por pruebas válidas o tardías.

Es CLAVE entender el funcionamiento, ya que para poder ganar la competencia y resolver las pruebas, las partes mas importantes son el precio de bloqueo y el fee de prioridad, ya que en boundless cuando agarras una prueba, la bloqueas para que solo tu y nadie mas la pueda resolver y ganarla, por eso el secxreto esta en lograr bloquear las pruebas antes que los demas.

Esto dependera de el fee que utilices y el precio al que estes dispuesto a bloquear, esto afectara tu rentabilidad y tus costos, los cuales seran diferentes en Testnets y en mainnet, ya que en esta segunda probablemente cuides más tu rentabilidad mientras que en la red de pruebas puedes ser más agresivo

## ☁️ Rentar GPU (opcional)
Podemos correrlo local, pero entiendo que muchos no tienen un equipo para esto e igual quieren participar, les dejo aqui algunas opciones para rentar un servidor con tarjeta grafica.

- Vast.ai es la opción recomendada:

Genera clave SSH: ssh-keygen

Usa template: Ubuntu o PyTorch (ya tiene CUDA y drivers)

- GPU Mart
Es otra opción que probamos para esto y tiene varias opciones de GPU y buen servicio al cliente.


Una vez que tenemos el servidor solo debemos acceder a el desde nuestro equipo.

- Conecta por SSH:

```bash
ssh -i ~/.ssh/id_rsa usuario@IP
```

Si estas en windows puedes usar Putty para acceder a tu servidor usando SSH. simplemente con tu iP del servidor usuario y clave podras hacerlo (buscate un video de como usar putty para ssh)
y ya estaras en el terminal de tu servidor alguilado! Felicidades

## 1️⃣ Instalar dependencias y Docker

Usaremos docker para la instalacion del nodo, asi que con estos comando actualizamos el sistema y descargamos las dependencias (software que necesitas para poder correr el nodo)

### 🔧 Dependencias básicas

- Verifica tu sistema operativo

```bash
lsb_release -a
```

- Detectar modelo gpu:
  
```bash
lspci | grep -i nvidia 
```

Esto debería mostrarte la línea con el modelo de tu tarjeta gráfica.

Por ejemplo:  ej: 03:00.0 VGA compatible controller: NVIDIA Corporation GP107 [GeForce GTX 1050 Ti] (rev a1)

Si no te devuelve eso, debemos borrar los drivers y luego instalar los correctos

```bash
sudo apt-get purge nvidia* -y
sudo apt-get autoremove -y
sudo apt-get autoclean
```

luego de borrarlos, reiniciamos:

```bash
sudo reboot
```

Luego vamos a ver los drivers para tu gpu.

```bash
sudo ubuntu-drivers devices
```

Deberia darte algo como:  

vendor : NVIDIA Corporation driver : nvidia-driver-575-open - third-party non-free driver : nvidia-driver-535-open - distro non-free driver : nvidia-driver-570-open - distro non-free

en mi caso procedo a instalar el primero 535

```bash
sudo apt update && sudo apt install -y nvidia-driver-575
```

```bash
sudo reboot
```

Y luego verifico usando:  

```bash
nvidia-smi 
```

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip -y
```

```bash
sudo apt update && sudo apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

### 🐳 Instalar Docker

Una ves listo, procedemos a descarga y probar Docker

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update && sudo apt upgrade -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

sudo docker run hello-world
sudo systemctl enable docker
sudo systemctl restart docker
```

---
## Verificar que Docker pueda usar el GPU

instala Nvidia runtime: 

```bash
sudo apt install nvidia-container-toolkit
sudo systemctl restart docker
```

 verifica la instalación, si no devuelve nada no esta instalado: 

 ```bash
 which nvidia-container-runtime
```

 📌 Paso: Verificar runtimes en Docker
Ejecuta:

```bash
cat /etc/docker/daemon.json
```

Asegúrate de que incluya algo como:

```bash
{
  "runtimes": {
    "bento": {
      "path": "/usr/local/sbin/bento-runc",
      "runtimeArgs": []
    },
    "nvidia": {
      "args": [],
      "path": "nvidia-container-runtime"
    }
  }
} 
```

* nvidia es necesario para que Docker use la GPU.
* bento es un runtime agregado automáticamente por Boundless (no se usa manualmente pero no debe borrarse).
  corre para verificar que docker puede usar tu gpu:

```bash
  docker run --rm --gpus all nvidia/cuda:12.3.2-base-ubuntu22.04 nvidia-smi
```

---

## 2️⃣ Instala Boundless

Ahora vamos a clonar el repositorio oficial de boundless

```bash
git clone https://github.com/boundless-xyz/boundless
cd boundless
git checkout release-0.10
```

Da permiso al script de instalacion automatico de Boundless: 

```bash
chmod +x ./scripts/setup.sh
```

corre el script:

```bash
sudo ./scripts/setup.sh
```

---

## 3️⃣ Descarga dependencias manuales:

# 1. Instalar rustup (gestor de Rust)

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

# 2. Cargar Rust en la terminal

```bash
. "$HOME/.cargo/env"
```

# 3. Actualizar rustup

```bash
rustup update
```

# 4. Instalar cargo (si no se instaló automáticamente)

```bash
sudo apt update && sudo apt install -y cargo
```

# 5. Verificar cargo

```bash
cargo --version
```

# 6. Instalar rzup (herramienta de RISC Zero)

```bash
curl -L https://risczero.com/install | bash
```

# 7. Cargar los cambios en bash

```bash
source ~/.bashrc
```

# 8. Verificar rzup

```bash
rzup --version
```

# 9. Instalar el toolchain de RISC Zero para Rust

```bash
rzup install rust
```

# 10. Instalar cargo-risczero

```bash
cargo install cargo-risczero
```
```bash
rzup install cargo-risczero
```

# 11. Actualizar rustup otra vez por si acaso

```bash
rustup update
```

# 12. Instalar Bento CLI

```bash
cargo install --git https://github.com/risc0/risc0 bento-client --bin bento_cli
```

# 13. Agregar cargo al PATH (solo si no está ya)

```bash
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

# 14. Verificar bento_cli

```bash
bento_cli --version
```

# 15. Instalar Boundless CLI

```bash
cargo install --locked boundless-cli
```
# 16. Verificar boundless-cli

```bash
boundless -h
```

si cargo da error verificar:   

```bash
sudo nano /etc/apt/sources.list.d/nvidia-container-toolkit.list
```
 
 deberia estar solo esta activa:  deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://nvidia.github.io/libnvidia-container/stable/deb/$(ARCH) /

```bash 
sudo apt update
```
```bash
sudo apt install -y cargo
```
 Verificar entorno limpio. Correr em cd boundless:  

```bash
just broker down || true
just broker clean || true
```
 
 dará warn por no haber configurado .env aun  probar bento: just bento
 si corre todo bien.  
 
 - corre una prueba:
    
```bash
RUST_LOG=info bento_cli -c 32
```
---

## 4️⃣ Comprobar puertos utilizados

Con este comando podemos verificar si los puertos estan siendo utilizados para evitar confilctos, si no tienes ningun otro servicio, entonces no estaran utilizados.

```bash
netstat -tuln | grep -E '30303|8545|8546|8551|5052|9000'
```

---

## 5️⃣ Configurar .ENV file:
En este archivo pondremos las configuraciones para arrancar el servicio depende de en que red quieres minar:

```bash
nano .env
```
Pega esto en tu archivo ENV dependiendo de la red cambiaran los valores, por ejemplo tu private key siempre sera el mismo indiferentemente de la red, pero el RPC debera ser el especifico para cada red, tambien los valores de verifier, market y set verifier, son las direcciones oficiales de boundless que varian segun la red y aqui puedes ver las oficiales:

El order stream es donde buscara las ordenes y tambien varia segun la red.

- https://docs.beboundless.xyz/developers/smart-contracts/deployments

```bash
PRIVATE_KEY=tu_clave_privada_sin_0x
RPC_URL=<tu rpc de sepolia>

VERIFIER_ADDRESS=0x925d8331ddc0a1F0d96E68CF073DFE1d92b69187
BOUNDLESS_MARKET_ADDRESS=0x13337C76fE2d1750246B68781ecEe164643b98Ec
SET_VERIFIER_ADDRESS=0x7aAB646f23D1392d4522CFaB0b7FB5eaf6821d64

ORDER_STREAM_URL="https://eth-sepolia.beboundless.xyz/"
```

Luego estamos listos para arrancar y probar con:

```bash
just broker up
```

Ver logs: 

```bash
docker compose logs -f
```
Logs del broker:
```bash
docker compose logs -f broker
```

 para bajar el servicio usamos 
 
 ```bash
 just broker down
```
---


## 6️⃣ Configuramos el Compose:

Aqui debemos ajustar el segment size y el kecck según recomendación oficial de boundless aqui: 

https://docs.beboundless.xyz/provers/performance-optimization#finding-the-maximum-segment_size-for-gpu-vram

bajamos y borramos el servicio
```bash
just broker down
just broker clean
```

cargamos variables del env

```bash
set -a
source .env
set +a
```

### 🔍 Ver contenedores:
Con este comando podemos ver los contenedores que estan ahora corriendo, y ver si estan funcionando bien, o si se estan reiniciando.

```bash
docker ps
```

### 🔍 Redis warn de overcommit:

da un warning por 
WARNING Memory overcommit must be enabled!  debemos:

1. Abre el archivo de configuración del sistema:
sudo nano /etc/sysctl.conf
  
2. Al final del archivo, agrega esta línea:

vm.overcommit_memory = 1
  
3. Guarda y cierra con CTRL + O, luego ENTER, y CTRL + X.

4. Aplica el cambio sin reiniciar (opcional):
    
sudo sysctl -w vm.overcommit_memory=1

5. (Recomendado) Reinicia tu máquina:

sudo reboot
 
✅ Esto eliminará ese warning la próxima vez que inicies Redis.
 

---

## 7️⃣ Stake USDC

ahora debemos hacer stake de usdc para oder empezar a probar! 

```bash
source .env  
boundless account deposit-stake 30
```

### 🧠 modifica y optimizar broker.toml según setup para ser mas eficiente (avanzado)


¿Listo para usar tu nodo Sepolia para pruebas y desarrollo? 🚀  

- dudas: @inbestprogram

https://x.com/inbestprogram
