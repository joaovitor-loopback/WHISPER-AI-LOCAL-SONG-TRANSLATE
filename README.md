# Song Translate

Baixa o áudio de vídeos do YouTube, transcreve com [faster-whisper](https://github.com/SYSTRAN/faster-whisper) (detecção automática de idioma) e traduz sempre para **português brasileiro (PT-BR)**, usando **DeepL** (online) ou **Argos Translate** (offline).

Roda 100% em CPU, sem necessidade de GPU.

## Funcionalidades

- Download de áudio via `yt-dlp` (uma URL ou lote a partir de arquivo `.txt`)
- Transcrição local com `faster-whisper` (modelos `tiny` a `large-v3`)
- Detecção automática do idioma da música
- Tradução para PT-BR com DeepL (melhor qualidade) ou Argos (offline)
- Processamento em lote: erro em uma música não interrompe as demais
- Saída em dois arquivos por música: transcrição original e tradução

## Requisitos

| Item | Detalhe |
|------|---------|
| Python | 3.12 recomendado (3.14 pode dar problema com o `argostranslate`) |
| ffmpeg | Obrigatório, precisa estar no `PATH` |
| Deno | Opcional, só se o yt-dlp reclamar de JS runtime |
| Internet | Necessária na primeira execução e para baixar do YouTube |
| RAM | ~8 GB confortável para o modelo `small` |

## Instalação

### 1. Instalar Python e ffmpeg (Windows)

```powershell
winget install Python.Python.3.12
winget install Gyan.FFmpeg
```

Feche e reabra o terminal, depois confira:

```powershell
py -3.12 --version
ffmpeg -version
```

No Linux/macOS, instale o ffmpeg pelo gerenciador de pacotes (`apt install ffmpeg`, `brew install ffmpeg`).

### 2. Clonar o repositório e criar o ambiente virtual

```powershell
git clone https://github.com/SEU_USUARIO/SEU_REPOSITORIO.git
cd SEU_REPOSITORIO
py -3.12 -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Se o PowerShell bloquear a ativação do venv:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

No Linux/macOS: `python3 -m venv venv && source venv/bin/activate`.

### 3. (Opcional) Configurar a chave do DeepL

Crie um arquivo `.env` na raiz do projeto:

```
DEEPL_API_KEY=sua_chave_aqui
```

> **Nunca** faça commit do `.env`. Ele já deve estar no `.gitignore`.

## Uso

Uma música, com Argos (offline):

```bash
python song_translate.py --urls "https://youtu.be/XXXX" --engine argos --model small
```

Uma música, com DeepL:

```bash
python song_translate.py --urls "https://youtu.be/XXXX" --engine deepl
```

Várias músicas (um link por linha em `lista.txt`, linhas com `#` são ignoradas):

```bash
python song_translate.py --urls-file lista.txt --engine argos --model small
```

### Opções

| Opção | Descrição | Padrão |
|-------|-----------|--------|
| `--urls` | Uma ou mais URLs do YouTube | - |
| `--urls-file` | Arquivo `.txt` com uma URL por linha | - |
| `--engine` | `deepl` (online) ou `argos` (offline) | `deepl` |
| `--deepl-key` | Chave da API DeepL (ou use `DEEPL_API_KEY`) | - |
| `--model` | Modelo do Whisper: `tiny`, `base`, `small`, `medium`, `large-v3` | `small` |
| `--output-dir` | Pasta de saída | `saida` |
| `--keep-audio` | Mantém os `.mp3` baixados | desligado |

## Saída

Para cada música, em `saida/`:

```
<nome>_original.txt   # transcrição no idioma original
<nome>_pt-br.txt      # tradução para PT-BR
```

Ao final, o script mostra um resumo com sucessos e falhas.

## Primeira execução

- O modelo do Whisper é baixado do Hugging Face (`small` ≈ 500 MB) e fica em cache.
- O Argos baixa o pacote do idioma detectado → português na primeira vez que o vê. Depois disso, a tradução funciona offline.

## Solução de problemas

| Erro | Causa e solução |
|------|-----------------|
| `WinError 2` ao baixar | O `yt-dlp.exe` não está no PATH. O script já chama `python -m yt_dlp`, então confirme que `pip install yt-dlp` rodou no venv ativo. |
| `ffprobe and ffmpeg not found` | Instale o ffmpeg (`winget install Gyan.FFmpeg`) e reabra o terminal. |
| `No supported JavaScript runtime` (warning) | Geralmente inofensivo. Se faltar formato, instale o Deno (`winget install DenoLand.Deno`) e atualize o yt-dlp (`pip install -U yt-dlp`). |
| `No module named 'faster_whisper'` | O venv não está ativo, ou as dependências foram instaladas em outro Python. Ative o venv e rode `pip install -r requirements.txt`. |
| `unexpected keyword argument 'metadata_errors'` | Incompatibilidade entre `faster-whisper` e `av`. O script contorna isso decodificando o áudio via ffmpeg. |
| `did not find executable at ...python.exe` | O venv foi copiado de outra máquina. Apague a pasta `venv` e crie de novo. |
| `argostranslate` falha ao instalar | Use Python 3.12 (`py -3.12 -m venv venv`). |

## Estrutura do projeto

```
.
├── song_translate.py
├── requirements.txt
├── README.md
├── .env            # local, não versionado
├── lista.txt       # opcional, lote de URLs
└── saida/          # gerado, não versionado
```

## Aviso

Use apenas com conteúdo que você tem direito de baixar e processar, respeitando os termos de uso do YouTube e os direitos autorais das obras. As traduções automáticas são aproximadas e podem errar gírias e expressões poéticas.

## Licença

Defina a licença do projeto (por exemplo, MIT) e adicione um arquivo `LICENSE`.
