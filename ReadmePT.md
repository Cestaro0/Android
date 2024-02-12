Aqui estão as notas sobre Android, programação e engenharia reversa.

O documento aborda os seguintes tópicos:
- Arquitetura do Android
- Engenharia Reversa com IDA PRO

## Arquitetura do Android
Primeiro, é necessário entender que a plataforma Android é baseada no kernel do Linux. Veja a imagem abaixo.

<img src="https://github.com/Cestaro0/Android/assets/99103680/6a73d8cd-4008-4e16-9dba-d4e1175cf083"/>

Essa imagem cobre várias camadas, e vamos discutir cada uma.

### Kernel do Linux
Como mencionado anteriormente, o Android é baseado no kernel do Linux, que fornece várias funcionalidades, como níveis de segurança, permissões e gerenciamento de usuários.

### HAL (Camada de Abstração de Hardware)
A HAL é a interface padrão para a implementação de hardware no Android. Ela é independente dos drivers de nível mais baixo.

### Tempo de Execução do Android
Esta parte é responsável pela execução dos aplicativos Android. Cada programa é executado em sua própria máquina virtual no formato .DEX (Dalvik Executable Format). A compilação do aplicativo pode ocorrer de duas maneiras: AOT (Ahead-Of-Time) e JIT (Just-In-Time).

### Bibliotecas Nativas C/C++
Esta parte, que considero mais interessante (já que adoro engenharia reversa), envolve muitos componentes no Android criados em código nativo (C/C++). Também podemos criar nossos próprios arquivos .so em C++ para aplicativos Android.

### API Java
Semelhante às bibliotecas nativas C/C++, a API Java utiliza pacotes Java para GUI ou funcionalidades de aplicativos com características exclusivas para o Android.

### Aplicativos do Sistema
O Android vem com um conjunto básico de aplicativos para e-mail, mensagens, calendário, navegação na web, contatos e muito mais.

## Engenharia Reversa
Este tópico é mais detalhado porque adoro engenharia reversa e é minha área de especialização. Vamos abordar o seguinte:

### ADB (Android Debug Bridge)
O ADB (Ponte de Depuração do Android) é uma interface de linha de comando que permite a comunicação direta e o controle de dispositivos Android conectados a um computador host. Essa ferramenta versátil permite que os desenvolvedores acessem recursos avançados do sistema operacional Android e realizem várias tarefas essenciais, como instalar/desinstalar aplicativos, transferir arquivos, diagnosticar problemas, simular eventos de toque, etc. Ele funciona via USB ou TCP. Os comandos mais usados são:

```
adb root
adb shell
adb push
adb pull
adb reboot
adb reboot bootloader
```

### ROOT
Como o Android é baseado no kernel do Linux, a linha de comando é extensivamente usada. Com permissões de usuário, "root" é o nível com todas as permissões (similar ao sudo). Para fazer root em um dispositivo, é necessário conceder acesso a "img.boot" (uma imagem modificada para acesso root) específica para o nosso dispositivo e usar a ferramenta "fastboot". Após conectar o dispositivo ao PC, use `adb reboot bootloader` para inicializar o dispositivo no modo de recuperação e, em seguida, use `fastboot boot "boot.img"`. Depois disso, nosso dispositivo está rooteado.

### APK
APK é a extensão dos aplicativos Android. Para acessar um arquivo APK em um dispositivo conectado, use `adb push "caminho para o aplicativo"`; então, no PC, o APK estará acessível como "base.apk". A estrutura do arquivo .apk é a seguinte:

```
  .apk
    |- extensão para bibliotecas C/C++
    |            |- libname.so
    |- classes .dex
    |- .xml
```

### JNI (Interface Nativa Java)
Agora, vamos falar sobre JNI (Java Native Interface). Para simplificar, é uma maneira que a Oracle criou para o Java se comunicar com o C++. Também existe o JNA, mantido por uma comunidade do GitHub (mas não vamos discuti-lo aqui). Abaixo está uma imagem de exemplo:

<img src="https://github.com/Cestaro0/Android/assets/99103680/f9ac3384-ea7c-4331-9fcb-a09f59a3aeb4">

Aqui, você pode ver que o JNI atua como intermediário entre a JVM e o .so em C++, onde executa o código Java, que por sua vez chama funções C++. O .so é criado com uma mistura de Java e C++ em um cabeçalho JNI. Para mais detalhes, consulte [este link](https://github.com/Cestaro0/How-To-Use-JNI).

### Análise com IDA
#### Estático
Ao desempacotar o arquivo .apk, podemos encontrar as pastas JNI .so e analisá-las. Engenharia reversa de arquivos .so é relativamente simples, mas um plugin é necessário:

- jni_all.h

Na arquitetura arm64-v8a (x64), não há problema em reconstruir estruturas. Portanto, você frequentemente encontrará algo como:

```c++
void foo(__int64 a1, __int64 a2, __int8* a3)
{
 wchar_t* a4;
 a4 = (wchar_t*)(**a1 + 123)(a1, a3, 0)
}
```

A análise pode ser complicada, mas não se preocupe; basta clicar na variável e pressionar "y" para exibir:

```c++
__int64 a1
```

Modifique para:

```c++
JNIEnv* a1
```

E funcionará como mágica:

```c++
a4 = (*env)->GetStringUTFChars(a3, 0)
```

No x32, é mais complicado, pois não aceita essa modificação, então você deve usar o cabeçalho. No IDA, acesse Arquivo > Carregar Arquivo > Analisar arquivo de cabeçalho C e escolha o `jni_all.h` baixado [deste link](https://gist.github.com/jcalabres

/bf8d530b3f18c30ca6f66388357b1d91) antes de usá-lo em x64. Observação: O cabeçalho não funcionará se houver qualquer proteção no arquivo .so.

#### Depuração / Dinâmico
Depurar aplicativos Android no IDA Pro envolve converter o código executável do aplicativo, presente no arquivo APK, em um formato legível e estruturado. Com o arquivo .so resultante, que contém o código em formato C/C++, o analista pode abrir a biblioteca no IDA Pro, explorar sua estrutura, identificar funções e entender o fluxo de execução. A capacidade de definir pontos de interrupção e pausar a execução do aplicativo em momentos específicos permite explorar a memória e os registradores do dispositivo durante a execução.

Preparação no IDA:
Carregue a biblioteca que você vai analisar no IDA e selecione os pontos de interrupção desejados. Depois disso, passaremos para a parte em que usamos o ADB.

Preparação com o ADB:
Na pasta do IDA Pro, você encontrará a pasta "dbgsrv". Dentro dela, você encontrará "android_server" (para telefones celulares armeabi-v7a x32) e "android_server64" (para telefones celulares arm64-v8a x64), de acordo com a arquitetura do seu dispositivo. Mova um dos arquivos para a pasta onde o ADB está. Depois disso, abra o terminal dentro da pasta do ADB e execute os seguintes comandos, com o telefone conectado ao PC via cabo USB com depuração USB ativada no dispositivo:

```
C:\platform-tools\> adb devices

C:\platform-tools\> adb push <android_server> /data/local/tmp

C:\platform-tools\> adb shell
```

Agora, no terminal do dispositivo:
```
android:/ $ su

android:/ # chmod 755 /data/local/tmp/<android_server>

android:/ # ./data/local/tmp/<android_server>

Depois disso, minimize este terminal e abra outro do ADB; vamos redirecionar as portas para que possamos depurar com o IDA:

C:\platform-tools\> adb forward tcp:23946 tcp:23946
```

Iniciando a depuração:
No IDA Pro, na guia "Debugger>Select debugger", selecione "Remote ARM Linux/Android debugger" e clique em OK.

Agora, na guia "Debugger>Process options...", no "Hostname", você vai digitar 127.0.0.1 e, no "Port", você vai inserir o mesmo número gerado no comando anterior. Depois disso, clique em OK.

Pronto, basicamente todas as configurações foram feitas e tudo está pronto para iniciar a depuração. Com o aplicativo que você deseja depurar aberto no seu dispositivo Android, vá para o IDA Pro na guia "Debugger>Attach to process..." para acessar os processos em execução no seu dispositivo, encontre o processo do aplicativo que deseja depurar. A partir daí, ele congelará o aplicativo com a execução pausada; pressione F9 e, em seguida, continue com o aplicativo. Quando ele atingir seu ponto de interrupção, pausará e você poderá depurar usando o IDA Pro (Lembre-se de manter "Usar depuração em nível de origem" ativado para a função de ponto de interrupção).
