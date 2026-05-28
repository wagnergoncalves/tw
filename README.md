# Apps Tecnoweb — APK

App nativo simples que abre `https://apps.tecnoweb.com.br/` em um WebView do Android.
Criado para resolver o problema do Chrome 56 (Android 7) que não consegue mais abrir o site
por TLS/handshake desatualizado.

## O que ele faz

- Abre `apps.tecnoweb.com.br` direto no `onCreate` em WebView nativo.
- WebView usa a engine do **Android System WebView** do aparelho — em totens Android 7
  isso geralmente é suficiente porque o WebView é atualizado independente do Chrome.
  Se o WebView do totem também estiver travado em uma versão antiga, atualize-o
  pela Play Store (pacote `com.google.android.webview`).
- Baixa APKs do site via `DownloadManager` (notificação nativa, salva em `Downloads/`).
- Ao terminar o download de um `.apk`, abre o instalador do sistema automaticamente.
- Links externos (fora de `tecnoweb.com.br`) abrem no navegador do sistema.
- `tel:`, `mailto:`, `whatsapp:` etc. são tratados pelos apps correspondentes.
- Botão Voltar navega o histórico do WebView.
- minSdk 24 → roda em **Android 7.0+**.

## Build — Opção 1: GitHub Actions (sem instalar nada)

1. Crie um repositório no GitHub.
2. Faça push deste projeto inteiro.
3. Vá em **Actions** → o workflow "Build APK" roda sozinho.
4. Baixe os artefatos `AppsTecnoweb-debug` ou `AppsTecnoweb-release`.

O APK de **debug** pode ser instalado direto no totem para teste.
O APK de **release** vem **não assinado** — para distribuição é necessário assinar
(veja seção "Assinatura" abaixo).

## Build — Opção 2: Android Studio

1. Abra a pasta no Android Studio (Hedgehog 2023.1+ ou superior).
2. Aguarde sincronização do Gradle (vai baixar Gradle 8.7 e SDK Android 34).
3. Menu **Build → Build Bundle(s) / APK(s) → Build APK(s)**.
4. APK fica em `app/build/outputs/apk/debug/app-debug.apk`.

## Build — Opção 3: Linha de comando

Pré-requisitos: JDK 17, Android SDK com platform 34 e build-tools 34.

```bash
# Gera o wrapper (uma vez)
gradle wrapper --gradle-version 8.7

# Build debug
./gradlew assembleDebug
# → app/build/outputs/apk/debug/app-debug.apk

# Build release (não assinado)
./gradlew assembleRelease
```

## Instalação no totem

Como o totem provavelmente não tem Play Store ou o site também não abre nele,
copie o APK por USB / pen drive / scp e instale com:

```bash
adb install -r app-debug.apk
```

Ou pelo gerenciador de arquivos do totem, abrindo o `.apk` direto.

**Atenção:** ative "Fontes desconhecidas" / "Instalar apps de fontes desconhecidas"
nas configurações do Android antes.

## Assinatura (para release de produção)

```bash
# 1. Gera keystore (uma vez, GUARDE BEM)
keytool -genkey -v -keystore tecnoweb.keystore -keyalg RSA -keysize 2048 \
  -validity 10000 -alias tecnoweb

# 2. Assina
$ANDROID_HOME/build-tools/34.0.0/apksigner sign \
  --ks tecnoweb.keystore \
  --out AppsTecnoweb.apk \
  app/build/outputs/apk/release/app-release-unsigned.apk

# 3. Verifica
$ANDROID_HOME/build-tools/34.0.0/apksigner verify AppsTecnoweb.apk
```

## Personalização rápida

- **URL inicial:** `MainActivity.java` → constante `START_URL`.
- **Domínio permitido (fica no WebView):** `MainActivity.java` → `ALLOWED_HOST`.
- **Nome do app:** `res/values/strings.xml`.
- **Ícone:** substitua os PNGs em `res/mipmap-*/ic_launcher.png` (o atual é um placeholder "TW").
- **Cor de fundo do ícone:** regere os ícones ou troque pelos seus.
- **versionCode / versionName:** `app/build.gradle`.

## Tamanho do APK

Com minify e resource shrinking habilitados, o release fica em ~1.5-2 MB.
O debug fica em ~3-4 MB.
