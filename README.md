~~perdoem meus erros gramaticais~~  
~~estou com um teclado estrangeiro e esta tudo trocado aqui~~  

~~caso algo esteja errado, de alguma forma, nao seja um chato. abre o PR e a gente corrige~~

# Android.bp

No build system do Android os arquivos .bp descrevem  
como um artefato deve ser construido e onde deve ser instalado.

Ele e auto explicativo. Ele tem um nome que deve ser unico,   
algumas configuracoes mais genericas do que compilar, de qual assinatura usar  
e se ele vai ter acesso as *platform_apis*, que sao escondidas do SDK  
padrao e nao podem ser capturadas nem via reflection:

> Ler: [Non-SDK interfaces](https://developer.android.com/guide/app-compatibility/restrictions-non-sdk-interfaces)  

# Nao bastando (como nada nessa vida basta)

Descrever um bp nao faz magicamente que seu app seja instalado  
no seu produto de desejo.  
Afinal existem varios produtos. Varias variantes.  
Por tanto, atraves do nome do .bp podemos dizer **onde** queremos que ele seja instalado.  
No meu caso, estou usando o **Android 11**, na branch **android-11.0.0_r39**.  
Isso pode ser visto abaixo (arquivo encontrado apos voce realizar a inicializacao do **repo**):

```
        <default revision="refs/tags/android-11.0.0_r39"
                remote="aosp"
                sync-j="4" />
```
