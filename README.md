# lxqt-avf-android
Requeriments: Android 16+
Debian ARM64 via AVF + LXQt no Termux:X11

Objetivo

Executar o LXQt instalado dentro de uma VM Debian ARM64, criada pelo Android Virtualization Framework (AVF), usando o Termux:X11 como servidor gráfico.

Arquitetura:

Android 16
│
├── Termux
│   └── Termux:X11
│       └── servidor X :0
│
└── AVF
    └── Debian ARM64
        └── LXQt

Neste setup, o Termux não executa o LXQt.

O LXQt continua rodando dentro do Debian da VM. O Termux:X11 apenas fornece a tela/servidor X para as aplicações gráficas.

---

Pré-requisitos

- Android 16
- Termux
- Termux:X11
- Debian ARM64 rodando via AVF
- LXQt instalado no Debian
- Conectividade de rede entre o Debian e o Android
- IP do Android acessível pela VM


1. Iniciar o Termux:X11

No Termux:

termux-x11 :0 -listen tcp -ac &

O que significam as opções?

:0

Usa o display X11 número 0.

-listen tcp

Permite que clientes X11 se conectem pela rede TCP.

-ac

Desativa temporariamente a autenticação de acesso do X11.

Isso foi necessário neste setup porque o Debian conseguiu alcançar o servidor X, mas recebeu:

Authorization required, but no authorization protocol specified

O "-ac" resolveu essa autenticação.

Importante

Não execute novamente:

termux-x11 :0 ...

se o servidor já estiver rodando.

Caso apareça:

server already running
Cannot establish any listening sockets

significa simplesmente que já existe um X server usando o ":0".

Para reiniciar:

pkill termux-x11

e depois:

termux-x11 :0 -listen tcp -ac &

---

2. Entrar no Debian AVF

Entre normalmente na VM, por exemplo via SSH:

ssh usuario@IP_DA_VM

Não importa qual usuário Debian você utiliza, desde que ele tenha acesso ao ambiente gráfico.

---

3. Definir o DISPLAY

Dentro do Debian:

export DISPLAY=10.46.116.168:0

Isso informa aos programas gráficos:

«"O servidor X está no Android, no endereço 10.46.116.168, display 0."»

---

4. Iniciar o LXQt

Ainda dentro do Debian:

dbus-run-session startlxqt

O LXQt deverá aparecer na janela do Termux:X11.

---

Processo completo

Depois que tudo estiver configurado, o procedimento fica simplesmente:

Termux

termux-x11 :0 -listen tcp -ac &

Debian AVF

export DISPLAY=10.46.116.168:0
dbus-run-session startlxqt

Pronto.

---

Se o Termux:X11 já estiver aberto

Não é necessário iniciar outro servidor X.

Se você executar novamente:

termux-x11 :0

e receber:

server already running

ignore. O servidor já está ativo.

---

Problemas encontrados

"Could not connect to display"

Exemplo:

qt.qpa.xcb: could not connect to display

Verifique se o Termux:X11 foi iniciado com:

termux-x11 :0 -listen tcp -ac &

E no Debian:

export DISPLAY=10.46.116.168:0

---

"Authorization required"

Exemplo:

Authorization required, but no authorization protocol specified

Neste setup, iniciar o X server com:

termux-x11 :0 -listen tcp -ac &

resolveu o problema.

---

"server already running"

Exemplo:

Cannot establish any listening sockets
server already running

Isso significa que o X server já está rodando.

Para reiniciar:

pkill termux-x11
termux-x11 :0 -listen tcp -ac &

---

Avisos do Termux:X11

Durante a inicialização podem aparecer mensagens relacionadas a:

mali_kbase
xkbcomp
XF86AudioBassBoost

Por exemplo:

Errors from xkbcomp are not fatal to the X server

Esses avisos não impediram o funcionamento do LXQt neste setup.

---

Observação de segurança

O parâmetro:

-ac

desativa o controle de acesso do X server.

Isso é conveniente para um ambiente local de teste, mas não é uma configuração ideal para expor um servidor X a uma rede não confiável.

O setup atual funciona porque estamos usando o Android/VM em um ambiente controlado.

Uma configuração futura pode substituir o "-ac" por autenticação X11 adequada.

---

Resultado

O resultado final é:

┌──────────────────────────────┐
│          Android 16          │
│                              │
│  ┌────────────────────────┐  │
│  │      Termux:X11        │  │
│  │       Display :0       │  │
│  └───────────▲────────────┘  │
│              │               │
│              │ TCP / X11     │
│              │               │
│  ┌───────────┴────────────┐  │
│  │       AVF / VM         │  │
│  │                        │  │
│  │    Debian ARM64        │  │
│  │         │              │  │
│  │       LXQt             │  │
│  └────────────────────────┘  │
└──────────────────────────────┘

O LXQt roda no Debian ARM64 da VM.

O Termux:X11 fornece apenas o servidor gráfico.

Não é necessário instalar LXQt no Termux e não é necessário utilizar "proot".