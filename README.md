#!/bin/bash

# --- Configurações Iniciais e Verificação de Permissões ---
if [ "$EUID" -ne 0 ]
  then echo "Por favor, execute como root (com sudo)."
  exit 1
fi

echo "Iniciando a instalação e configuração do Cloudflared..."
echo "---"

# 1. Adicionar a chave GPG pública do Cloudflare
echo "Passo 1/4: Adicionando a chave GPG do Cloudflare..."
mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-public-v2.gpg | tee /usr/share/keyrings/cloudflare-public-v2.gpg >/dev/null

if [ $? -eq 0 ]; then
    echo "Chave GPG adicionada com sucesso."
else
    echo "Erro ao adicionar a chave GPG. Abortando."
    exit 1
fi
echo "---"

# 2. Adicionar o repositório Cloudflare ao APT
echo "Passo 2/4: Adicionando o repositório Cloudflare ao APT..."
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-public-v2.gpg] https://pkg.cloudflare.com/cloudflared any main' | tee /etc/apt/sources.list.d/cloudflared.list

if [ $? -eq 0 ]; then
    echo "Repositório adicionado com sucesso."
else
    echo "Erro ao adicionar o repositório. Abortando."
    exit 1
fi
echo "---"

# 3. Atualizar a lista de pacotes e instalar o cloudflared
echo "Passo 3/4: Atualizando a lista de pacotes e instalando o cloudflared..."
apt-get update && apt-get install -y cloudflared

if [ $? -eq 0 ]; then
    echo "Cloudflared instalado com sucesso."
else
    echo "Erro ao instalar o Cloudflared. Abortando."
    exit 1
fi
echo "---"

# 4. Instalar o serviço Cloudflared com o token fornecido
# O token aqui é: eyJhIjoiYTVmY2EzMDRjZTQ3NjYxOTc4YmM5NDgyYWY1NTUyNzUiLCJ0IjoiMzdkNjM3MTgtYmY2ZS00ZTk3LThkZjUtN2VkYTdhMGI3ODU1IiwicyI6IlpUZzFZV1ZsWldJdE4yUXpNeTAwT1RnMExXSmpPRGN0TmpFM09EZ3dZMkptWmpNeSJ9
TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVhMGJmYzA0ZTc0NTRiMjdiYmQyN2Q3NGQ2ZGMzYjI2IiwidG9rZW4iOiIzN2Q2MzcxOC1iZjZlLTRlOTctOGRmNS03ZWRhN2EwYjc4NTUiLCJzIjoielRneklaVldsWldJdE4yUXpNeTAwT1RnMExXSmpPRGN0TmpFM09EZ3dZMkptWmpNeSJ9" # Substitua com o token real se for diferente do exemplo
echo "Passo 4/4: Instalando o serviço Cloudflared com o token..."
cloudflared service install "$TOKEN"

if [ $? -eq 0 ]; then
    echo "Serviço Cloudflared instalado e iniciado com sucesso."
else
    echo "Atenção: Houve um erro ao instalar/iniciar o serviço Cloudflared. Verifique as configurações."
fi
echo "---"

echo "Instalação e configuração do Cloudflared concluídas."
