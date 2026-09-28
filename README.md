# WebOS-KolibriOS-
KolibriOS for WebOS. Tested on WebOS 7.6.0

# KolibriOS WebOS

Emulador x86 no teu LG webOS TV. Boota o [KolibriOS](https://kolibrios.org/) — um sistema operacional open source escrito inteiramente em assembly, com GUI completa, que ocupa ~1.4 MB e corre com 16 MB de RAM.

Download the `.ipk` from GitHub and install using ares-install.

## How to Use

1. Download do ficheiro `.ipk` pré-compilado deste repositório
2. Ativa o **Developer Mode** na tua TV e liga-a via webOS CLI
3. Instala a app usando:

```bash
ares-install --device tv2 com.kolibri.webos_1.0.0_all.ipk
```

4. Abre a app pelo menu da TV (ou via `ares-launch --device tv2 com.kolibri.webos`)

A app abre o KolibriOS em full screen na tua TV.

## Features

- **KolibriOS completo** com GUI, mouse, teclado, editor de texto, visualizador de imagens, jogos
- **Sem dependência de rede** — o emulador e o sistema estão embutidos no `.ipk`
- **Interface gráfica** a 640×480 (qualquer profundidade de cor)
- **Tamanho reduzido** — o `.ipk` final fica em ~4 MB

## Especificações Técnicas

| Parâmetro | Valor |
|---|---|
| Emulador | [v86](https://github.com/copy/v86) (x86 → WebAssembly) |
| Sistema convidado | KolibriOS (imagem de disquete, 1.4 MB) |
| `memory_size` | 16 MB |
| `vga_memory_size` | 300 KB |
| `fda` | `kolibri.img` |
| Tamanho do `.ipk` | ~4 MB |

## Estrutura

```
.
├── appinfo.json          # manifesto da app webOS
├── icon.png              # ícone 80x80
├── index.html            # instancia o V86Starter
├── lib/
│   ├── v86-bundle.js     # bundle do v86 (IIFE, sem imports ES)
│   └── v86-wasm-b64.js   # v86.wasm em base64 (contorna CORS em file://)
└── vm/
    └── kolibri.img       # imagem do KolibriOS
```

## Build (a partir do código-fonte)

```bash
# instalar dependências do v86 e gerar o bundle
cd lib
npm init -y
npm install v86 esbuild

cat > bundle-entry.js << 'INNER'
import { V86Starter } from "./node_modules/v86/build/index.js";
import { seabios, vgabios } from "./node_modules/v86/build/binaries.js";
window.V86Starter = V86Starter;
window.v86Binaries = { seabios, vgabios };
INNER

npx esbuild bundle-entry.js --bundle --format=iife --outfile=v86-bundle.js

# gerar o WASM em base64 (contorna restrição de file:// no webOS)
node -e "
const fs = require('fs');
const wasm = fs.readFileSync('node_modules/v86/build/v86.wasm');
fs.writeFileSync('v86-wasm-b64.js',
  'window.v86WasmBase64 = \"' + wasm.toString('base64') + '\";');
"

# empacotar
cd ..
ares-package --no-minify .
```

## Limitações

- O webOS impõe um teto de memória por processo. Em TVs com pouca RAM livre, o OOM killer do sistema pode encerrar a app.
- **Sem persistência entre boots**: as alterações feitas dentro do KolibriOS (resolução, tema, ficheiros) ficam em RAM e perdem-se ao fechar a app.
- **Seleção de vídeo no boot**: o bootloader do KolibriOS mostra o menu de seleção de vídeo por alguns segundos. Para escolher, clica no canvas e aperta uma seta antes do timeout — senão ele escolhe a opção padrão sozinho.

## Créditos

- [v86](https://github.com/copy/v86) — Fabrice Bellard e contribuidores (BSD-2-Clause)
- [KolibriOS](https://kolibrios.org/) — comunidade KolibriOS (GPLv2)
- Este app — apenas cola os dois dentro do sandbox do webOS

## License

O código deste app (HTML, JS de cola, README) é MIT.
O v86 é BSD-2-Clause. O KolibriOS é GPLv2.
