~~perdoem meus erros gramaticais~~  
~~estou com um teclado estrangeiro e esta tudo trocado aqui~~  

~~caso algo esteja errado, de alguma forma, nao seja um chato. abre o PR e a gente corrige~~

# Cade o gradle?
Se voce ja programou para Android, sabe, ou deveria saber, que para que um APK  
seja gerado e feito uso do [gradle](https://gradle.org/).  
Mas nao quer dizer que um APK nao possa ser gerado sem ele.  
Em resumo, para que um APK seja compilado, assim como quaquer artefato de qualquer  
framework especifico, e necessario algum tipo de SDK, compilador, ou qualquer coisa  
que produza um binario que faca sentido quando carregado pela plataforma.  
Para o Android nao e diferente.  

## Um APK e como um JAR. (~~e as motos como os jetskis~~)  
Sua unica diferenca e que ele possui arquivos, como fotos, XML etc.  
De maneira breve, para gerar um APK e necessario compilar suas classes .java ou .kt  
que vao gerar arquivos .class (bytecode). Esses devem ser transformados para arquivos .dex, para  
rodar na maquina virtual do Android (ART).  
A palavra *dex* significa algo como *dalvik executable*. Dalvik era a antiga VM do Android.  
A ART abre esse arquivo *.dex* em tempo de instalacao e faz o cache deles para *.odex*, que otimiza  
as intrucoes da VM para o device em especifico, tornando o tempo de instalacao demorado, mas o de  
execucao rapido.  
Os outros arquivos, conhecidos como recursos, devem ser compilados de alguma maneira e serem  
encapsulados dentro desse APK.  
Para isso e usado uma ferramenta chamada [aapt2](https://developer.android.com/studio/command-line/aapt2).

# Android.bp
No build system do Android ([soong](https://android.googlesource.com/platform/build/soong/+/refs/heads/master/README.md))
os arquivos .bp descrevem  
como um artefato deve ser construido e onde deve ser instalado.  
Ele e auto explicativo.
Ele tem um nome que deve ser unico, algumas configuracoes mais genericas do que compilar,  
de qual assinatura usar e se ele vai ter acesso as *platform_apis*, que sao escondidas do SDK  
padrao e nao podem ser capturadas nem via reflection:

> Ler: [Non-SDK interfaces](https://developer.android.com/guide/app-compatibility/restrictions-non-sdk-interfaces)  

# Nao bastando (como nada nessa vida basta)
Descrever um bp nao faz magicamente que seu app seja instalado no seu produto de desejo.  
Afinal existem varios produtos. Varias variantes.  
Por tanto, atraves do nome do .bp podemos dizer **onde** queremos que ele seja instalado.  
No meu caso, estou usando o **Android 11**, na branch **android-11.0.0_r39**.  
Isso pode ser visto abaixo (arquivo encontrado apos voce realizar a inicializacao do **repo**):

```
        <default revision="refs/tags/android-11.0.0_r39"
                remote="aosp"
                sync-j="4" />
```

## Produtos, produtos e produtos  
~~e intrinseca necessidade de criar solucoes para os problemas que criamos~~

Como descrito acima, e necessario dizer **onde**.  

No meu caso, usando a **branch** descrita acima, temos os seguintes produtos (e alguns mais irrelevantes):  

```
Lunch menu... pick a combo:
     1. aosp_arm-eng
     2. aosp_arm64-eng
     3. aosp_blueline-userdebug
     4. aosp_blueline_car-userdebug
     5. aosp_bonito-userdebug
     6. aosp_bonito_car-userdebug
     7. aosp_bramble-userdebug
     8. aosp_car_arm-userdebug
     9. aosp_car_arm64-userdebug
     10. aosp_car_x86-userdebug
     11. aosp_car_x86_64-userdebug
     12. aosp_cf_arm64_auto-userdebug
     13. aosp_cf_arm64_phone-userdebug
     14. aosp_cf_x86_64_phone-userdebug
     15. aosp_cf_x86_auto-userdebug
     16. aosp_cf_x86_phone-userdebug
     17. aosp_cf_x86_tv-userdebug
     18. aosp_coral-userdebug
     19. aosp_coral_car-userdebug
     20. aosp_crosshatch-userdebug
     21. aosp_crosshatch_car-userdebug
     22. aosp_flame-userdebug
     23. aosp_flame_car-userdebug
     24. aosp_redfin-userdebug
     25. aosp_sargo-userdebug
     26. aosp_sunfish-userdebug
     27. aosp_trout_arm64-userdebug
     28. aosp_trout_x86-userdebug
     29. aosp_x86-eng
     30. aosp_x86_64-eng
```

Como eu quero compilar para que seja executado na minha maquina, a opcao 30  
me foi mais adequada. Suponho que codigos x86_64 possam ser executados direto na KVM  
sem a necessidade de uma traducao, diferente da arquitetura arm ou arm64, que precisam ser "traduzidas".  
Nao sou um expert. Seja voce. Pesquise.

> Ler: [KVM](https://www.linux-kvm.org/page/Main_Page)  