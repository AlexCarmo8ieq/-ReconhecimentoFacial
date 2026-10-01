# Reconhecimento de texto em imagens (OCR)

Projeto de exemplo que aplica OCR a quatro imagens com condições diferentes
(texto limpo, números e valores, texto girado com ruído, etiqueta borrada e de baixo contraste)
e mede o quanto o resultado se aproxima do texto correto.

> **Nota de transparência:** o ambiente em que este exemplo foi montado não acessa o Azure,
> então os resultados em `output/` foram gerados com o **Tesseract** (`ocr_local_tesseract.py`).
> O script `ocr.py` faz o mesmo com o **Azure AI Vision** e grava no mesmo formato.
> Na sua entrega, rode `ocr.py` com a sua chave e use os seus resultados e prints do Azure.

## Estrutura

```
├── inputs/                      # imagens usadas + gabarito.json (texto correto de cada uma)
├── output/                      # .txt (texto), .json (confiança por palavra) e metricas.json
├── prints/                      # imagens com as caixas de texto reconhecidas
├── ocr.py                       # OCR com Azure AI Vision
├── ocr_local_tesseract.py       # OCR local (usado nos resultados deste exemplo)
├── gerar_imagens_exemplo.py     # cria as imagens de teste
└── readme.md
```

## Processo

1. **Imagens:** `gerar_imagens_exemplo.py` cria 4 imagens em `inputs/` com texto conhecido, salvo em `inputs/gabarito.json`.
2. **OCR:** o script lê cada imagem e grava em `output/` o texto (`.txt`) e as palavras com a confiança (`.json`).
   Com Azure, `ocr.py` usa `ImageAnalysisClient` com `VisualFeatures.READ`.
3. **Avaliação:** o texto reconhecido é comparado ao gabarito (similaridade de 0 a 1) e a confiança média é calculada.
4. **Prints:** as caixas reconhecidas são desenhadas sobre cada imagem em `prints/`.

## Imagens, resultados e prints

| Imagem | Desafio | Similaridade com o gabarito | Confiança média |
|---|---|---|---|
| `01_placa_seguranca.png` | Texto grande, alto contraste, acentos | 1,000 | 0,959 |
| `02_recibo.png` | Várias linhas, números, valores em R$ | 1,000 | 0,930 |
| `03_texto_acentos_girado.png` | Inclinação de 3° e ruído | 0,983 | 0,881 |
| `04_etiqueta_baixo_contraste.png` | Baixo contraste e desfoque | 0,979 | 0,913 |

**01 – Placa de segurança** (`output/01_placa_seguranca.txt`)

![placa](prints/01_placa_seguranca_caixas.png)

**02 – Recibo** (`output/02_recibo.txt`)

![recibo](prints/02_recibo_caixas.png)

**03 – Texto girado com ruído** (`output/03_texto_acentos_girado.txt`)

![texto girado](prints/03_texto_acentos_girado_caixas.png)

**04 – Etiqueta de baixo contraste** (`output/04_etiqueta_baixo_contraste.txt`)

![etiqueta](prints/04_etiqueta_baixo_contraste_caixas.png)

## Insights

- **Condições da imagem importam mais que o tipo de texto.** As duas imagens limpas foram reconhecidas sem nenhum erro; as duas degradadas tiveram pequenas falhas.
- **Acentos foram bem reconhecidos** (`Relatório`, `manutenção`, `ATENÇÃO`, `Feijão`) ao usar o idioma português junto com o inglês.
- **Ruído gera "texto fantasma".** Na imagem 03 apareceram um `;` e uma `,` que não existem na imagem, ambos com baixa confiança (o `;` ficou em 0,47, bem abaixo dos 0,92 a 0,97 das palavras reais).
- **A confiança por palavra aponta onde olhar.** Na imagem 04 o número de série saiu como `8F3K-290D-4471` em vez de `8F3K-29QD-4471` (o `Q` virou `0`), e foi justamente a palavra com a menor confiança da imagem (0,82).
- **Códigos e números de série são o ponto fraco**: não existe dicionário que ajude a "adivinhar" `Q` versus `0`, então eles precisam de validação.

## Possibilidades

- **Campo e manutenção:** ler placas, etiquetas e números de série de equipamentos por foto, em vez de digitar à mão.
- **Documentos:** extrair dados de notas, recibos e relatórios para planilhas ou bancos de dados.
- **Controle de qualidade:** marcar automaticamente para revisão humana as palavras com confiança abaixo de um limite (por exemplo 0,85).
- **Validação por regra:** conferir o formato do resultado (CNPJ, datas, padrão de série) para pegar erros como `Q` versus `0`.
- **Pré-processamento:** corrigir inclinação, contraste e ruído antes do OCR tende a melhorar os casos difíceis.

## Como executar

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt

python gerar_imagens_exemplo.py  # cria as imagens de teste

# Opção A: Azure AI Vision
cp .env.example .env             # preencha VISION_ENDPOINT e VISION_KEY
python ocr.py

# Opção B: local, com Tesseract
python ocr_local_tesseract.py
```
