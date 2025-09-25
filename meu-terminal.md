# 🖥️ Meu Emulador de Terminal

Este repositório tem como objetivo ensinar você a configurar o seu **emulador de terminal** para ficar igual ao que eu uso no meu dia a dia.

> **Importante:**
> Toda a minha configuração é baseada no **Bash**, o interpretador padrão do Debian.
> Eu **não uso Zsh**, nem troco para outros interpretadores.

---

## 📦 Instalando o URxvt

O emulador que eu uso é o **URxvt** (rxvt-unicode), que está disponível nos repositórios oficiais do Debian.
Para instalar, execute:

```bash
sudo apt update
sudo apt install rxvt-unicode
```


Por padrão, o URxvt é bem simples e visualmente pobre. Vamos melhorar isso!

---

## ⚙️ Configurando o URxvt

Crie ou edite o arquivo `~/.Xresources`:

```bash
nano ~/.Xresources
```

E adicione estas configurações:

```text
URxvt*foreground: white
URxvt*background: black
URxvt.font: xft:0xProtoNerdFontMono-Regular:size=20
URxvt*scrollBar: true
URxvt*transparent: true
URxvt*shading: 15

```

---

## 🔤 Instalando a Fonte

Baixe e instale a fonte **0xProto Nerd Font**:

```bash
mkdir -p ~/.local/share/fonts
cd ~/.local/share/fonts
wget https://github.com/ryanoasis/nerd-fonts/releases/download/v3.4.0/0xProto.zip
unzip 0xProto.zip
fc-cache -fv
```

---

## 🔄 Aplicando as Configurações

Após editar o `~/.Xresources`, aplique as mudanças:

```bash
xrdb ~/.Xresources
```

Se o pacote xrdb não existir, faça a instalação do x11-xserver-utils

```bash
sudo apt install x11-xserver-utils
```

Agora é só abrir o URxvt — seu terminal estará personalizado! ✅

---

## 🎨 Deixando a Listagem de Arquivos Bonita

Para exibir ícones e uma listagem estilosa, instale o **lsd**:

```bash
sudo apt install lsd
```

Adicione o alias no seu `~/.bashrc` para substituir o `ls`:

```bash
echo "alias ls='lsd'" >> ~/.bashrc
source ~/.bashrc
```

Agora sempre que digitar `ls`, será usado o `lsd` com ícones e cores.

---

## 🖋️ Personalizando o Prompt (PS1)

Para deixar o prompt igual ao meu, adicione esta linha ao seu `~/.bashrc`:

```bash
PS1="\[\e[1;32m\]\342\224\214\342\224\200\[\e[1;32m\][\[\e[1;33m\]\u\[\e[1;37m\]@\[\e[1;33m\]\h\[\e[1;32m\]]\[\e[1;32m\]\342\224\200\[\e[1;32m\][\[\e[1;33m\]\w\[\e[1;32m\]]\[\e[1;32m\]\342\224\200[\[\e[1;37m\]\t\[\e[1;32m\]]\[\e[1;32m\]\342\224\200[\[\e[1;37m\]$(date +%a)\[\e[1;32m\]]\n\[\e[1;32m\]\342\224\224\342\224\200\342\224\200\342\225\274\[\e[1;37m\] \$ \[\e[0m\]"
```

> Dica: se quiser algo diferente, recomendo o [script do Professor Kretcheu](https://github.com/kretcheu/devel/blob/6169f932217385fdafb3ddc90b571737a2f3c88d/prompt), que permite escolher entre vários modelos de PS1.

---

Se o bash estiver bloqueando o clipboard você pode desativar a proteção adicionando no seu ~/.bashrc

```bash
if [[ $TERM =~ xterm.*|rxvt.* ]]; then
    printf '\e[?2004l'
fi
```




## ✅ Resultado Final

Com essas etapas, você terá:

* URxvt com **fonte Nerd Font**, **cores personalizadas** e **pseudo-transparência**
* Listagem de arquivos bonita com **ícones coloridos (lsd)**
* Um **prompt estilizado** com informações úteis (usuário, host, diretório, hora e dia)

Agora seu terminal estará muito mais produtivo e elegante! 🎉

