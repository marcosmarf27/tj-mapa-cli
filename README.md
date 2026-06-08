# tj-mapa

**Índice navegável de processos judiciais para o seu agente de IA.**

O `tj-mapa` lê os autos de um processo em Markdown e devolve um **mapa estrutural**: cada peça
(petição, despacho, decisão, sentença, certidão, ofício, mandado…) com **página e linha
exatas**, tipo e resumo. Assim o agente vai **direto ao trecho certo** em vez de reler o
processo inteiro — rápido e a custo de centavos.

Parte do ecossistema **TecJustiça**: OCR jurídico ([ocr.tecjustica.com](https://ocr.tecjustica.com/))
→ **tj-mapa** → MCP ([mcp.tecjustica.com](https://mcp.tecjustica.com/)) — usados em conjunto com o Claude Code.

---

## 1. Baixar

Pegue o arquivo do seu sistema na página de **[Releases](https://github.com/marcosmarf27/tj-mapa-cli/releases/latest)**:

| Sistema | Arquivo |
|---|---|
| **Windows** (x86-64) | `tj-mapa_windows_amd64.exe` |
| **Linux / WSL** (x86-64) | `tj-mapa_linux_amd64` |
| **Skill do Claude Code** | `tj-mapa.skill` |

## 2. Instalar (deixa `tj-mapa` disponível em qualquer terminal)

Rode **uma vez, pelo caminho do arquivo baixado** — ainda não precisa estar no PATH; é o
próprio `install` que o coloca lá:

**Windows (PowerShell):**
```powershell
.\tj-mapa_windows_amd64.exe install
```
**Linux / WSL:**
```bash
chmod +x ./tj-mapa_linux_amd64
./tj-mapa_linux_amd64 install
```

Abra um **novo terminal** e pronto — `tj-mapa` funciona em qualquer lugar. Instalação
**por usuário** (sem admin). Para remover: `tj-mapa uninstall`.

## 3. Configurar a chave (OpenRouter)

```
tj-mapa config set        # cola a chave e tecla Enter
```

Pegue a sua chave em [openrouter.ai/keys](https://openrouter.ai/keys). Também funciona a
variável de ambiente `OPENROUTER_API_KEY` ou um arquivo `.env` no diretório de trabalho.

> Dá para instalar e já configurar a chave de uma vez: `... install --key sk-or-...`

## 4. Usar

```
tj-mapa -in processo.md -formato ambos
```

Gera **`mapa.json`** (para o agente consumir) e **`mapa.md`** (tabela legível, com custo e tempo
no topo). A entrada é o Markdown do processo (saída do OCR, uma página por marcador
`## Página N`). Para só a estrutura, mais rápido: adicione `-sem-enriquecer`. Todas as opções:
`tj-mapa -h`.

## 5. Skill do Claude Code (opcional)

Baixe `tj-mapa.skill` (é um zip) e extraia na pasta de skills do Claude Code:

```bash
# WSL / Linux
unzip tj-mapa.skill -d ~/.claude/skills/
```
```powershell
# Windows
Expand-Archive .\tj-mapa.skill -DestinationPath "$env:USERPROFILE\.claude\skills\"
```

Reinicie o Claude Code. A skill traz o binário embutido e dispara quando você menciona
**"tj-mapa"** ou pede o **"mapa do processo"**.

---

*Software proprietário da TecJustiça. Distribuído como binário compilado; o código-fonte não é público.*
