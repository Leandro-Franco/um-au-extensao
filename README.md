<p align="center">
  <img src="icons/icon-128.png" alt="um - AU" width="88">
</p>

<h1 align="center">um - AU — captura de atendimentos veterinários</h1>

<p align="center">
  Extensão de navegador (Manifest V3) que lê o atendimento aberto num sistema clínico
  veterinário, deixa a pessoa revisar os dados e envia para uma planilha do Google Sheets
  via <b>n8n</b>.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Manifest-V3-informational" alt="Manifest V3">
  <img src="https://img.shields.io/badge/Firefox-140%2B%20%C2%B7%20Android%20142%2B-informational" alt="Firefox 140+">
  <img src="https://img.shields.io/badge/JavaScript-sem%20depend%C3%AAncias-informational" alt="JavaScript sem dependências">
  <img src="https://img.shields.io/badge/licen%C3%A7a-MIT-green" alt="MIT">
</p>

---

## O problema

Numa clínica veterinária, cada atendimento era digitado duas vezes: no sistema
clínico (SimplesVet) e numa planilha de controle. A extensão elimina a segunda
digitação **sem mudar a planilha nem o sistema**, e sem tirar o controle humano:
nada sai do navegador sem um clique.

## Como funciona

```
SimplesVet (aba do navegador)
  └─ content.js  ── lê o DOM ──► painel flutuante: revisar e editar
                                        │ clique em "Enviar"
                                        ▼
  background.js (script de fundo) ── POST JSON ──► webhook n8n ──► nova linha no Google Sheets
```

1. A veterinária abre o atendimento normalmente e clica em **Capturar**.
2. A extensão preenche paciente, prontuário, espécie, sexo, tutor e patologia.
3. Ela confere, corrige o que precisar, escolhe o tipo de atendimento e clica em **Enviar**.
4. O n8n recebe o JSON e acrescenta a linha na planilha.

## Diferenciais técnicos

| | |
|---|---|
| **Humano no controle** | Máquina de estados explícita (`IDLE → AGUARDANDO_CAPTURA → CAPTURADO → ENVIANDO → ENVIADO/ERRO`) e nenhum envio automático: capturar e enviar são dois cliques separados. |
| **Fora do CSP da página** | O `fetch` para o webhook roda no script de fundo da extensão, não na página do sistema — a política de segurança do site hospedeiro não bloqueia o envio. |
| **Página dinâmica** | `MutationObserver` acompanha a navegação interna do sistema (SPA) e religa os observadores sem duplicar *listeners*. |
| **Atalho de prontuário** | Ao escolher "Consulta clínica geral", aplica um modelo de exame no editor do prontuário (iframe do TinyMCE). |
| **Rascunho que sobrevive** | A captura em andamento fica em `storage.local`: recarregar a página não perde o que foi editado. |
| **Envio com prazo** | `AbortController` corta a requisição em 10 s e mostra o erro em vez de travar o painel. |
| **Privacidade declarada** | Usa as `data_collection_permissions` do Firefox (dados de saúde e pessoais), exigidas para extensões que tratam esse tipo de dado. |
| **Zero dependências** | JavaScript puro, sem build. O pacote final tem só o código e os ícones. |

## Configuração

Nada vem preenchido: cada instalação aponta para o próprio n8n.

1. **n8n** — crie um workflow com um nó *Webhook* (POST) seguido de um nó
   *Google Sheets → Append row*. Se quiser, valide o campo `token` do corpo antes
   de gravar.
2. **`manifest.json`** — troque `https://seu-n8n.exemplo.com/*` em
   `host_permissions` pelo endereço do seu n8n, e o `gecko.id` por um id seu.
3. **`content.js`** — ajuste `PAYING_VET` para o nome da veterinária como aparece
   no usuário logado do sistema.
4. **Popup** — depois de instalar, clique no ícone da extensão e informe a URL do
   webhook e o token. Eles ficam só no armazenamento local do navegador.

### Payload enviado

```json
{
  "token": "…",
  "mes": "NOVEMBRO",
  "data": "06/11/2026",
  "prontuario": "12345",
  "paciente": "Nina",
  "especie": "Felina",
  "sexo": "Fêmea",
  "tutor_responsavel": "Nome do tutor",
  "tipo_atendimento": "SIMPLES",
  "veterinario_responsavel": "NOME SOBRENOME",
  "patologia": "…"
}
```

Todos os campos, menos o `token`, são obrigatórios: o painel não envia se faltar algum.

## Rodar e empacotar

Com o [`web-ext`](https://extensionworkshop.com/documentation/develop/getting-started-with-web-ext/) da Mozilla:

```bash
npx web-ext run      # abre um Firefox de teste com a extensão carregada
npx web-ext lint     # valida manifest e código
npx web-ext build    # gera o .zip em web-ext-artifacts/
```

## Roadmap

- Testes automatizados dos seletores principais
- Log local de envios e indicador de status mais completo
- Suporte a mais de uma veterinária por instalação

## Sobre esta versão pública

Este repositório é um **retrato público** da extensão, que é mantida num
repositório privado e desenvolvida desde junho de 2026 (versão 1.0.2). A URL
do webhook, o token, o id de publicação e o nome da veterinária foram retirados e
viraram configuração. O projeto não tem vínculo com o SimplesVet.

## Licença

[MIT](LICENSE) © 2026 Leandro Franco

---

### In English

**um - AU** is a Manifest V3 browser extension (Firefox desktop and Android) that
reads the appointment currently open in a veterinary clinic system, lets the vet
review and edit the captured fields, and posts them to an **n8n** webhook that
appends a row to Google Sheets. It removes double data entry without changing the
spreadsheet or the clinic system, and nothing is sent without an explicit click.
Highlights: an explicit state machine, network calls from the background script to
stay outside the host page's CSP, `MutationObserver` handling of SPA navigation,
draft persistence in `storage.local`, request timeouts, and zero dependencies.
This is a public snapshot of a private repository; all endpoints and tokens are
configuration.
