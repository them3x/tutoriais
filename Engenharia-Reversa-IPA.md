## ENGENHARIA REVERSA EM APLICATIVOS IOS
<hr>

#### Preparando ambiente

Antes vamos precisa instalar o default-jdk para podermos executar a ferramenta de engenharia reversa GHYDRA
```
apt install default-jdk
```

Agora podemos baixar a ferramenta desenvolvida pela [NSA a GHydra](https://github.com/NationalSecurityAgency/ghidra/)

```
messias@debian:~/codigo$ wget https://github.com/NationalSecurityAgency/ghidra/releases/download/Ghidra_11.1.2_build/ghidra_11.1.2_PUBLIC_20240709.zip
messias@debian:~/codigo$ unzip ghidra_11.1.2_PUBLIC_20240709.zip
```
<hr>

#### Mão na massa

Vou ultilzar o aplicativo [CoinZa](https://github.com/izmcm/iOS-App-Security/raw/master/CoinZa.ipa) disponibilizado no Github do [izmcm](https://github.com/izmcm/iOS-App-Security/) 
<br><br>
Ao analizar um aplicativos IOS teremos um pacote IPA, que na verdade trata-se apenas de um zip responsavel por manter tudo necessario para executar o aplicativo em questão

```
messias@debian:~/codigo$ ls -l
.rw-r--r-- messias messias  14 MB Tue Jul 16 17:02:48 2024  CoinZa.ipa
drwxr-xr-x messias messias 4.0 KB Tue Jul  9 12:19:08 2024  ghidra_11.1.2_PUBLIC
.rw-r--r-- messias messias 403 MB Tue Jul 16 12:54:39 2024  ghidra_11.1.2_PUBLIC_20240709.zip
messias@debian:~/codigo$
messias@debian:~/codigo$
messias@debian:~/codigo$
messias@debian:~/codigo$ file CoinZa.ipa 
CoinZa.ipa: Zip archive data, at least v1.0 to extract, compression method=store
messias@debian:~/codigo$ unzip CoinZa.ipa
```
Ao descompactar o pacote IPA, temos toda estrutura do pacote necessario para executar o aplicativo
```
messias@debian:~/codigo$ cd Payload/
messias@debian:~/codigo/Payload$ ls
 CoinZa.app
messias@debian:~/codigo/Payload$ cd CoinZa.app/
messias@debian:~/codigo/Payload/CoinZa.app$ ls
 _CodeSignature        CoinZa                                                 Info.plist
 AppIcon60x60@2x.png   coinza-c7e97-firebase-adminsdk-ok3f8-df3457e3e8.json   PkgInfo
 AppIcon60x60@3x.png   CreateWalletCollectionViewCell.nib                     WalletCollectionViewCell.nib
 Assets.car            Frameworks                                             WhitepaperCollectionViewCell.nib
 Base.lproj            GoogleService-Info.plist                              
messias@debian:~/codigo/Payload/CoinZa.app$ 
```
Aqui podemos ver toda estrutura de arquivos do aplicativo em questão.
- _CodeSignature (Arquivos de assinatura digital)
- Assets.car (Arquivo de recursos visuais compilados)
- Base.jproj (Arquivos de localização, arquivos de layout em geral)
- Coinza (Arquivo executavel)
- coinza-c7e97-firebase-adminsdk-ok3f8-df3457e3e8.json (Aqui json, parece conter informações administrativas do firebase)
- Frameworks (Arquivos de frameworks para funcionalidades adicionais do app)
- GoogleService-Info.plist (Arquivo de configuração para serviços Google)
- Info.plist (Arquivo de propiedades compilado)
- PkgInfo (Informações basicas do aplicativo)
- *.nib (Arquivos XIB compilados)

```
messias@debian:~/codigo/Payload/CoinZa.app$ file CoinZa 
CoinZa: Mach-O universal binary with 2 architectures: [armv7:\012- Mach-O armv7 executable, flags:<NOUNDEFS|DYLDLINK|TWOLEVEL|PIE>] [\012- arm64]
```
O que realmente nos interessa aqui é o arquivo CoinZa que como podemos ver, trata-se de um binario universal Mach-O compilado para 2 arquiteturas
- ARMV7 (Arquitetura mais antiga dos processadores da apple)
- ARM64 (Arquitetura mais nova)
  <br><br><br>
Diferentes de arquivos DEX que tratam-se de bytecodes para o interpretador Dalvik, o binario Mach-O (geralmente escrito em [Objective-C](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Introduction/Introduction.html) ou [Swift](https://www.swift.org/documentation/) de fato esta compilado para ser executado no hardware do aparelho, o que significa que não existe uma forma simples de trazer o codigo de maquina para uma linguagem alto nivel, podemos tentar trazer isso para um assembly e com ajuda de decompiladores trazer isso para um C, porem devemos estar ciente de que a decompilação apesar de facilitar a leitura pode não representar de forma exata o assembly puro do executavel. Para isso vamos usar o Ghydra
```
messias@debian:~/codigo$ cd ghidra_11.1.2_PUBLIC/
messias@debian:~/codigo/ghidra_11.1.2_PUBLIC$ ls
 bom.json   Extensions   ghidraRun       GPL       licenses   support
 docs       Ghidra       ghidraRun.bat   LICENSE   server    
messias@debian:~/codigo/ghidra_11.1.2_PUBLIC$ ./ghidraRun
```
Com o GHydra podemos criar um novo projeto e tentar analizar o nosso aplicativo.
