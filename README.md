# Hermes_Agent_de_IA_Autonomo_no_Docker 

> ⚠️ **Setup não-oficial.** Este repositório contém apenas configurações Docker
> (docker-compose, README, comandos) para rodar o **[Hermes Agent](https://github.com/NousResearch/hermes-agent)**
> da [Nous Research](https://nousresearch.com) no Windows com Telegram e tambem outras 15+ plataformas de mensageria.
>
> O Hermes Agent é um projeto open source (MIT License) da Nous Research.
> Toda a tecnologia do agente, skills, gateway e features pertence a eles.
> Aqui você só encontra a "casca" Docker para facilitar o uso no Windows 11.
>
> 📖 Documentação oficial: https://hermes-agent.nousresearch.com/docs/
> 💻 Repositório oficial: https://github.com/NousResearch/hermes-agent

---

Este pacote é o caminho correto para o que você descreveu:

```text
Seu PC Windows 11
  ↓
Docker Desktop
  ↓
Container com Hermes Agent oficial
  ↓
Gateway do Hermes
  ↓
Telegram / Discord / WhatsApp etc.
```

A ideia é rodar o **Hermes Agent oficial dentro do Docker**

---

## 1. O que este pacote faz

Ele prepara:

- `docker-compose.yml` usando a imagem oficial `nousresearch/hermes-agent:latest`
- pasta `hermes-data/` para memória/configuração do Hermes (salva API keys, sessões, skills)
- pasta `workspace/` para arquivos que você permite o agente acessar
- comandos para configurar Telegram com BotFather

---

## 2. Estrutura

```text
hermes_docker_jarvis_base/
├─ docker-compose.yml
├─ .env.example          (opcional — só para HERMES_UID/GID)
├─ README.md
├─ COMANDOS.md
├─ hermes-data/           (config, memória, sessões, skills — criado pelo setup)
└─ workspace/             (pasta de trabalho liberada para o agente)
```

---

## 3. Como funciona o isolamento

O container NÃO enxerga seu PC inteiro.

Ele só enxerga as pastas montadas no `docker-compose.yml`:

```yaml
volumes:
  - ./hermes-data:/opt/data
  - ./workspace:/workspace
```

Dentro do container:

```text
/opt/data   = configuração, memória, sessões, skills, API keys do Hermes
/workspace  = pasta de trabalho liberada por você
```

No seu Windows:

```text
hermes-data/
workspace/
```

O `network_mode: host` permite que o container use a rede do host diretamente (necessário para algumas plataformas como WhatsApp e Signal).

---

## 4. Primeiro passo: criar o bot no Telegram

No Telegram:

1. Abra `@BotFather`
2. Envie:

```text
/newbot
```

3. Escolha nome e username
4. Copie o token

Depois fale com:

```text
@userinfobot
```

Pegue seu Telegram User ID.

---

## 5. Arquivo `.env` da raiz — precisa?

O `.env` da raiz é **opcional**. O `hermes setup` salva tudo (API keys, Telegram token, modelo) dentro de `./hermes-data/.env` automaticamente.

O único motivo para criar o `.env` da raiz seria ajustar `HERMES_UID`/`HERMES_GID` (permissões do volume). O `docker-compose.yml` já usa o valor padrão `1000`, que funciona no Docker Desktop para Windows.

Se quiser criar mesmo assim:

```powershell
copy .env.example .env
```

---

## 6. Rodar setup inicial do Hermes dentro do Docker

Na pasta do projeto, rode:

```powershell
docker compose run --rm hermes setup
```

Esse comando abre o setup oficial do Hermes.

Ele vai perguntar:

- provedor/modelo de IA (ex: OpenRouter)
- API key do provedor
- Telegram token e User ID
- outras plataformas de mensageria
- preferências do agente

Tudo é salvo automaticamente em `./hermes-data/`. Não precisa editar `.env` manualmente.

---

## 7. Rodar o gateway do Hermes

Depois do setup:

```powershell
docker compose up -d
```

Agora pode rodar o comando para abrir o chat no Powershell (Ou então crie um alias para esse comando no Powershell):

```powershell
docker compose exec hermes /opt/hermes/.venv/bin/hermes
```

Ver logs:

```powershell
docker compose logs -f hermes
```

Parar:

```powershell
docker compose down
```

---

## 8. Testar no Telegram

Abra seu bot no Telegram e envie:

```text
/status
```

Depois teste:

```text
Oi, Hermes. Quem é você?
```

---

## 9. Dashboard opcional

O `docker-compose.yml` já vem com dashboard comentado.

Se quiser usar dashboard depois, descomente o bloco `dashboard`.

As portas ficam acessíveis diretamente via `network_mode: host`:

```text
http://localhost:9119   (dashboard)
http://localhost:8642   (gateway/API)
```

Se for usar só Telegram, nenhuma porta precisa ser exposta para a internet.

---

## 10. Tailscale: precisa colocar dentro do container?

Para Telegram, normalmente NÃO precisa.

O gateway do Hermes conversa com Telegram por conexão de saída.

O Tailscale é útil para você acessar remotamente:

- Docker Desktop
- dashboard local
- API do Hermes
- arquivos do workspace
- terminal do PC

Recomendação mais simples:

```text
Instale Tailscale no Windows 11, não dentro do container.
```

Assim você acessa o PC remotamente pelo IP do Tailscale.

Colocar Tailscale dentro do container é possível, mas aumenta complexidade e geralmente não é necessário no começo.

---

## 11. Segurança recomendada

Não use:

```env
GATEWAY_ALLOW_ALL_USERS=true
```

Use sempre:

```env
TELEGRAM_ALLOWED_USERS=SEU_ID
```

Não monte seu `C:\` inteiro.

Não faça isso:

```yaml
- C:\:/workspace
```

Faça isso:

```yaml
- ./workspace:/workspace
```

---

## 12. Atualizar Hermes

```powershell
docker compose pull
docker compose up -d
```

A pasta `hermes-data/` preserva suas memórias, sessões, skills e chaves.

---

## 13. Comandos rápidos (rodar no PowerShell normal, sem admin)

Antes de tudo, entre na pasta do projeto:

```powershell
cd C:\Users\pc\Downloads\hermes_docker_jarvis_base\hermes_docker_jarvis_base
```

### Sequência principal

```powershell
# 1. Primeira vez (configurar tudo: modelo, API keys, Telegram, etc.)
docker compose run --rm hermes setup

# 2. Subir o gateway em background
docker compose up -d

# 3. Para parar no dia a dia
docker compose down

# 4. Para voltar
docker compose up -d
```

### Comandos úteis

```powershell
# Ver logs em tempo real (Ctrl+C para sair sem parar)
docker compose logs -f hermes

# Chat interativo no terminal (sem Telegram)
docker compose run --rm hermes

# Diagnóstico de problemas
docker compose run --rm hermes doctor

# Configurar gateway (Telegram, Discord, etc.)
docker compose run --rm hermes gateway setup

# Atualizar para última versão
docker compose pull
docker compose up -d
```
