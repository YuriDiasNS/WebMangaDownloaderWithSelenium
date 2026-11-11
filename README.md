# Web Manga Downloader with Selenium

Baixador de capítulos de mangá em Python. A partir da URL de um capítulo, ele escolhe o *fetcher* adequado ao site, coleta os links das imagens com **Selenium**, baixa todas as páginas em paralelo e abre um **leitor de imagens** em Tkinter para ler o capítulo logo em seguida.

![Python 3.11](https://img.shields.io/badge/Python-3.11-3776AB?logo=python&logoColor=white)
![Selenium](https://img.shields.io/badge/Selenium-automação%20web-43B02A?logo=selenium&logoColor=white)
![Pillow](https://img.shields.io/badge/Pillow-validação%20de%20imagens-blue)

---

## Sumário

- [Visão geral](#visão-geral)
- [Funcionalidades](#funcionalidades)
- [Como funciona](#como-funciona)
- [Arquitetura](#arquitetura)
- [Leitor de imagens](#leitor-de-imagens)
- [Tecnologias](#tecnologias)
- [Instalação](#instalação)
- [Uso](#uso)
- [Estrutura do projeto](#estrutura-do-projeto)

---

## Visão geral

Muitos sites de leitura carregam as páginas dinamicamente, o que impede um simples `requests.get` de encontrar as imagens. Este projeto resolve isso com um navegador Chrome em modo *headless* controlado pelo Selenium, e separa cada site em um *fetcher* próprio. O download das imagens em si é feito à parte, de forma concorrente e com tentativas automáticas.

Os fetchers que acompanham o repositório (`ImageLinkFetcher_exemplo_1` e `ImageLinkFetcher_exemplo_2`) são **exemplos**, com domínios fictícios (`nome_do_site_1.biz` e `nome_do_site_2.net`), que mostram dois padrões diferentes de extração de links.

## Funcionalidades

- **Seleção automática de fetcher** pela URL informada (`LinkFetcherChooser`)
- **Coleta de links com Selenium**, usando Chrome headless e `webdriver-manager` para gerenciar o driver
- **Download paralelo** com `ThreadPoolExecutor` (10 threads por padrão)
- **Tentativas automáticas:** até 15 tentativas por imagem, com timeout de 10 s por requisição
- **Validação de imagens** com Pillow antes de gravar em disco
- **Nomes sequenciais** de arquivo (`0001.jpg`, `0002.jpg`, ...), que preservam a ordem das páginas
- **Relatório final** com a quantidade de downloads bem-sucedidos e a lista das URLs que falharam
- **Leitor integrado** (Tkinter), com tema escuro, zoom e navegação por blocos de imagens

## Como funciona

```
URL do capítulo
      │
      ▼
LinkFetcherChooser ──► escolhe o fetcher pelo domínio da URL
      │
      ▼
ImageLinkFetcher_* ──► abre a página com Selenium (Chrome headless)
      │                 e extrai os links das imagens
      ▼
ImageSaver ──────────► baixa as imagens em paralelo, valida e salva em
      │                 Mangas/<nome-do-capítulo>/0001.jpg, 0002.jpg, ...
      ▼
mangaReader.py ──────► abre o leitor de imagens (Tkinter)
```

1. O `main.py` recebe a URL do capítulo.
2. O `LinkFetcherChooser` compara o domínio com os sites suportados e devolve o fetcher correspondente. Se nenhum servir, exibe `URL não suportada.`
3. O fetcher carrega a página, espera 3 segundos e coleta os links das imagens.
4. O `ImageSaver` cria a pasta `Mangas/<último segmento da URL>/` e baixa cada imagem em uma thread.
5. Ao final, o `main.py` entra na pasta `Mangas` e executa o `mangaReader.py`.

## Arquitetura

| Arquivo | Responsabilidade |
| --- | --- |
| `main.py` | Ponto de entrada: orquestra fetcher, download e leitor |
| `LinkFetcherChooser.py` | Mapeia o domínio da URL para o fetcher correspondente |
| `SitesDownloaders/` | Um fetcher por site, responsável por extrair os links das imagens |
| `ImageSaver.py` | Download concorrente, validação, retentativas e gravação em disco |
| `Mangas/mangaReader.py` | Leitor de imagens em Tkinter |

### Contrato dos fetchers

Cada fetcher é uma classe que recebe a URL no construtor e expõe dois métodos:

```python
class ImageLinkFetcher_exemplo:
    def __init__(self, url): ...
    def fetch_image_links(self): ...   # navega até a página e coleta os links
    def get_image_links(self): ...     # devolve a lista de URLs das imagens
```

Os dois fetchers de exemplo mostram abordagens distintas:

| Fetcher | Estratégia |
| --- | --- |
| `ImageLinkFetcher_exemplo_1` | Lê o atributo `src` das tags `<img loading="lazy">` da página do capítulo |
| `ImageLinkFetcher_exemplo_2` | Lê o `data-src` das miniaturas em `#thumbnail-container` e converte cada URL para a da imagem em tamanho original |

### ImageSaver

| Parâmetro / comportamento | Valor |
| --- | --- |
| `max_threads` | 10 (configurável no construtor) |
| Tentativas por imagem | 15, com 1 s de espera entre elas |
| Timeout por requisição | 10 s |
| Pasta de destino | `Mangas/<último segmento da URL>/` |
| Nome dos arquivos | `0001.jpg`, `0002.jpg`, ... |

## Leitor de imagens

O `mangaReader.py` é um leitor simples em Tkinter com tema escuro:

1. Mostra uma janela de seleção com as pastas de capítulos existentes em `Mangas/`
2. Carrega as imagens da pasta escolhida, ordenadas numericamente
3. Exibe **10 imagens por vez** em uma lista com rolagem (roda do mouse)

| Botão | Ação |
| --- | --- |
| Carregar Próximas Imagens | Exibe o próximo bloco de 10 imagens |
| Voltar Imagens Anteriores | Volta ao bloco anterior |
| Zoom In / Zoom Out | Amplia ou reduz as imagens (fator de 1,5x) |
| Escolher Outra Pasta | Troca de capítulo sem reiniciar o programa |

## Tecnologias

| Biblioteca | Uso |
| --- | --- |
| Python 3.11 | Linguagem principal |
| Selenium | Automação do Chrome para carregar páginas dinâmicas |
| webdriver-manager | Download e gerenciamento do ChromeDriver |
| Requests | Download das imagens |
| Pillow (PIL) | Validação das imagens e renderização no leitor |
| Tkinter | Interface do leitor de imagens |
| `concurrent.futures` | Pool de threads para downloads paralelos |

## Instalação

Pré-requisitos: Python 3.11+ e Google Chrome instalado.

```bash
git clone https://github.com/YuriDiasNS/WebMangaDownloaderWithSelenium.git
cd WebMangaDownloaderWithSelenium

pip install selenium webdriver-manager requests pillow
```

O Tkinter já vem com a instalação padrão do Python no Windows e no macOS.

## Uso

1. Abra o `main.py` e defina a URL do capítulo:

   ```python
   url = "https://nome_do_site_1.biz/capitulo/3181/"
   ```

2. Execute:

   ```bash
   python main.py
   ```

3. As imagens são salvas em `Mangas/<nome-do-capítulo>/` e, ao terminar, o leitor é aberto.

Exemplo da saída no terminal ao final do download:

```
Tentativas bem-sucedidas: 24
Tentativas malsucedidas: 0
```

Para ler capítulos já baixados, sem baixar nada de novo, execute o leitor diretamente:

```bash
cd Mangas
python mangaReader.py
```

## Estrutura do projeto

```
.
├── main.py                          # ponto de entrada
├── LinkFetcherChooser.py            # escolhe o fetcher pela URL
├── ImageSaver.py                    # download paralelo e gravação das imagens
├── SitesDownloaders/
│   ├── ImageLinkFetcher_exemplo_1.py
│   └── ImageLinkFetcher_exemplo_2.py
└── Mangas/
    ├── mangaReader.py               # leitor de imagens (Tkinter)
    └── <capítulos baixados>/        # 0001.jpg, 0002.jpg, ...
```
