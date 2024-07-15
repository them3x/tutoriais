## ENGENHARIA REVERSA EM APLICATIVOS ANDROID
<hr>

#### Preparando ambiente
Antes de mais nada vamos precisar instalar alguns pacotes
- Java JDK
- flatpak
- jd-gui

Vamos usar os pacotes de desenvolvimento JDK para executar algumas ferramentas importantes, e o flatpak vamos usar para instalar o jd-gui, porem você tambem pode optar por [compilar o jd-gui](https://github.com/java-decompiler/jd-gui) manualmente, porem vou optar por instalar via flatpak.

```
apt install default-jdk flatpak
flatpak install jd-gui
```

Uma vez instalado o jd-gui, podemos listar nossos pacotes instados com

```
root@debian:# flatpak list
Nome                       ID de aplicativo                      Versão                     Ramo         Instalação
OBS Studio                 com.obsproject.Studio                 29.1.3                     stable       system
JD-GUI                     io.github.java_decompiler.jd-gui      1.6.6                      stable       system
Freedesktop Platform       org.freedesktop.Platform              freedesktop-sdk-23.08.21   23.08        system
Mesa                       org.freedesktop.Platform.GL.default   23.1.6                     22.08        system
Mesa (Extra)               org.freedesktop.Platform.GL.default   23.1.6                     22.08-extra  system
Mesa                       org.freedesktop.Platform.GL.default   24.1.3                     23.08        system
Mesa (Extra)               org.freedesktop.Platform.GL.default   24.1.3                     23.08-extra  system
openh264                   org.freedesktop.Platform.openh264     2.1.0                      2.2.0        system
Adwaita theme              org.kde.KStyle.Adwaita                                           6.4          system
KDE Application Platform   org.kde.Platform                                                 6.4          system
```
Podemos executar o jd-gui com o comando

```
flatpak run io.github.java_decompiler.jd-gui
```
Para facilitar tambem podemos configurar um alias

```
alias jd-gui="flatpak run io.github.java_decompiler.jd-gui"
```

JD-GUI é um utilitário gráfico independente que exibe códigos-fonte Java, vamos usa-lo para visualizar o codigo fonte dos APKs que vamos trabalhar.
<br><br>
Para trabalhar a um nivel mais baixo, vamos usar outras 2 ferramentas, o [APKTOOL](https://apktool.org/) e o [Baksmali](https://github.com/JesusFreke/smali?tab=readme-ov-file) tanto o APKTOOL quanto Baksmali vão nos servir para o mesmo proposito "extrair o codigo smali" do nosso APK alvo
<br><br>
Uma vez baixado, deveriamos ter 2 arquivos .jar

```
messias@debian:~/codigo$ ls -l
.rw-r--r-- messias messias  22 MB Sat Apr 27 12:59:23 2024 apktool_2.9.3.jar
.rw-r--r-- messias messias 1.2 MB Mon Jul 15 17:12:02 2024 baksmali-2.5.2.jar
.rw-r--r-- messias messias  10 KB Wed Jul 10 00:11:24 2024 codigo.apk
messias@debian:~/codigo$ 
```

Tambem vamos ultilizar uma ferramenta chamada [dex2jar](https://github.com/pxb1988/dex2jar) esta ferramenta responsavel por automatizar a conversão de um Dalvik Executable para um arquivo Java alto nivel legivel.

```
messias@debian:~/codigo$ wget https://github.com/pxb1988/dex2jar/releases/download/v2.4/dex-tools-v2.4.zip
--2024-07-15 17:19:35--  https://github.com/pxb1988/dex2jar/releases/download/v2.4/dex-tools-v2.4.zip
Resolvendo github.com (github.com)... 20.201.28.151
Conectando-se a github.com (github.com)|20.201.28.151|:443... conectado.
A requisição HTTP foi enviada, aguardando resposta... 302 Found
Localização: https://objects.githubusercontent.com/github-production-release-asset-2e65be/32313383/f27c4742-c6f9-483a-bf58-3b7c788edecb?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20240715%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240715T201935Z&X-Amz-Expires=300&X-Amz-Signature=e79505e162e8ec47d4c507da6cfd1ececa67d6242a72b73906be48eec3621423&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=32313383&response-content-disposition=attachment%3B%20filename%3Ddex-tools-v2.4.zip&response-content-type=application%2Foctet-stream [redirecionando]
--2024-07-15 17:19:35--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/32313383/f27c4742-c6f9-483a-bf58-3b7c788edecb?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20240715%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240715T201935Z&X-Amz-Expires=300&X-Amz-Signature=e79505e162e8ec47d4c507da6cfd1ececa67d6242a72b73906be48eec3621423&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=32313383&response-content-disposition=attachment%3B%20filename%3Ddex-tools-v2.4.zip&response-content-type=application%2Foctet-stream
Resolvendo objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.111.133, 185.199.110.133, 185.199.108.133, ...
Conectando-se a objects.githubusercontent.com (objects.githubusercontent.com)|185.199.111.133|:443... conectado.
A requisição HTTP foi enviada, aguardando resposta... 200 OK
Tamanho: 19105975 (18M) [application/octet-stream]
Salvando em: “dex-tools-v2.4.zip”

dex-tools-v2.4.zip            100%[===============================================>]  18,22M  55,7MB/s    em 0,3s    

2024-07-15 17:19:36 (55,7 MB/s) - “dex-tools-v2.4.zip” salvo [19105975/19105975]

messias@debian:~/codigo$ unzip dex-tools-v2.4.zip 
```
Uma vez baixado e descompactado o arquivo zip do repositorio do dex2jar teremos todas as ferramentas prontas para fazer engenharia reversa em um APK
<hr>

#### Mão na massa

uma vez com as ferramentas instaladas, trabalhar com nosso arquivo .apk, primeiro podemos "dezipar" nosso arquivo apk com o comando unzip
```
messias@debian:~/codigo$ unzip -d codigo codigo.apk 
Archive:  codigo.apk
  inflating: codigo/AndroidManifest.xml  
  inflating: codigo/resources.arsc   
  inflating: codigo/classes.dex      
   creating: codigo/META-INF/
  inflating: codigo/META-INF/MANIFEST.MF  
  inflating: codigo/META-INF/SIGNFILE.SF  
  inflating: codigo/META-INF/SIGNFILE.RSA  
messias@debian:~/codigo$ cd codigo/
messias@debian:~/codigo/codigo$ ls
AndroidManifest.xml  classes.dex  META-INF  resources.arsc
messias@debian:~/codigo/codigo$ 

```
Aqui temos todos os arquivos que compõe o nosso APK, entretanto os arquivos estão em formato byte-codes
```
messias@debian:~/codigo/codigo$ file AndroidManifest.xml classes.dex 
AndroidManifest.xml: Android binary XML
classes.dex:         Dalvik dex file version 035
```
Ou seja, não podemos trabalhar diretamente com esses arquivos pois isso na verdade se trata de arquivos que serão interpretado pelo Dalvik para executar funções no seu smarthphone
<br><br>
Porem podemos usar o apktool para extrair o codigo smali presente do arquivo .deX
```
messias@debian:~/codigo$ java -jar apktool_2.9.3.jar d codigo.apk -o codigo_decompilado
I: Using Apktool 2.9.3 on codigo.apk
I: Loading resource table...
I: Decoding file-resources...
I: Decoding values */* XMLs...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: /home/messias/.local/share/apktool/framework/1.apk
I: Regular manifest package...
I: Baksmaling classes.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...
messias@debian:~/codigo$ cd codigo_decompilado/
messias@debian:~/codigo/codigo_decompilado$ ls
AndroidManifest.xml  apktool.yml  original  res  smali
messias@debian:~/codigo/codigo_decompilado$ 
```
Tambem podemos usar o baksmali
```
messias@debian:~/codigo$ java -jar baksmali-2.5.2.jar d codigo.apk -o codigo_decompilado_baksmali
messias@debian:~/codigo$ cd codigo_decompilado_baksmali/
messias@debian:~/codigo/codigo_decompilado_baksmali$ ls
com
messias@debian:~/codigo/codigo_decompilado_baksmali$ cd com/
messias@debian:~/codigo/codigo_decompilado_baksmali/com$ ls
metasploit
messias@debian:~/codigo/codigo_decompilado_baksmali/com$ cd metasploit/
messias@debian:~/codigo/codigo_decompilado_baksmali/com/metasploit$ ls
stage
messias@debian:~/codigo/codigo_decompilado_baksmali/com/metasploit$ cd stage/
messias@debian:~/codigo/codigo_decompilado_baksmali/com/metasploit/stage$ ls
a.smali  c.smali  e.smali  g.smali             MainBroadcastReceiver.smali  Payload.smali
b.smali  d.smali  f.smali  MainActivity.smali  MainService.smali            
messias@debian:~/codigo/codigo_decompilado_baksmali/com/metasploit/stage$ 
```
Podemos ver certa diferença usando as 2 ferramentas, o apktool acaba nos fornecendo alguns arquivos a mais porem ambos nos fornecem o importante que é o codigo smali.
<br><br>
O [código Smali](https://github.com/JesusFreke/smali/wiki) é uma representação de baixo nível para executáveis Dalvik, semelhante ao assembly utilizado em executáveis ELF (Linux) e PE (Windows). No ambiente Android, o Java é compilado em bytecode específico que roda na Dalvik Virtual Machine (DVM), armazenado em arquivos .dex. O Smali, então, é como um assembly para esse bytecode, permitindo uma visualização e edição mais acessíveis do conteúdo dos arquivos .dex. Essa capacidade de edição é essencial para tarefas como análise de segurança, modificações de aplicativos ou engenharia reversa de software Android.
<br><br>
Trabalhar diretamente com código Smali pode ser bastante complexo. Para facilitar, a ferramenta dex2jar é usada para elevar o nível de abstração, similarmente ao que fazemos com bytecodes Python (.pyc) em análises de engenharia reversa. Com dex2jar, é possível converter os arquivos .dex (Dalvik Executable) de volta para código Java. Isso nos permite recuperar grande parte da estrutura original do código, tornando a análise e compreensão muito mais acessíveis. Esta capacidade é crucial para a análise de segurança e engenharia reversa em aplicações Android.
```
messias@debian:~/codigo$ bash dex-tools-v2.4/d2j-dex2jar.sh codigo.apk
dex2jar codigo.apk -> ./codigo-dex2jar.jar
messias@debian:~/codigo$ 
```
Agora podemos usar o jd-gui para visualizar o conteudo do arquivo codigo.jar, permitindo a visualização de toda estrutura java do nosso APK

```
messias@debian:~/codigo$ flatpak run io.github.java_decompiler.jd-gui codigo-dex2jar.jar
```
