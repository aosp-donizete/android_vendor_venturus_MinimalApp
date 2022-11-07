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
os arquivos .bp descrevem como um artefato deve ser construido e onde deve ser instalado.  
Ele e auto explicativo.  
Ele tem um nome que deve ser unico, algumas configuracoes mais genericas do que compilar,  
de qual assinatura usar e se ele vai ter acesso as *platform_apis*, que sao escondidas do SDK  
padrao e nao podem ser capturadas nem via reflection:

> Ler: [Non-SDK interfaces](https://developer.android.com/guide/app-compatibility/restrictions-non-sdk-interfaces)  

# Compilando  
Para compilar qualquer coisa dentro do framework, voce so precisa conhecer 1 comando:  
**mm module_name**  
E toda magica acontece.  
Como nosso modulo se chama "MinimalApp", basta digitar, depois de subir o ambiente do framework:
**mm MinimalApp**  
E provavel que voce obtenha algum erro relacionado ao arquivo **AndroidManifest.xml**.  
Nos conseguimos criar um APK sem nenhum *bytecode* dentro dele. Mas o manifest e sagrado.  
Para isso devemos criar um arquivo de manifest dentro do nosso modulo.

Algo como:

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.venturus.minimal.app">

    <application
        android:name="MinimalApp"
        android:label="MinimalApp"></application>
</manifest>
```

O meu modulo se encontra assim:  
```
.
├── Android.bp
├── AndroidManifest.xml
├── baguncinha.jpg
└── README.md
```

~~seja sensato, voce nao precisa da imagem do baguncinha~~

Depois disso voce conseguira compilar seu APK corretamente.

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
~~e a intrinseca necessidade de criar solucoes para os problemas que criamos~~

Como descrito acima, e necessario dizer **onde**.  

No meu caso, usando a **branch** descrita acima, temos os seguintes produtos:  
~~eu omiti o resto depois do 30~~  

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

Esse produto vem de alguma configuracao descrita na pasta:  
**/device**  
ou  
**/build/target/product**  

O framework Android nao e conhecido por ser bem documentado.  
Uma grande baguncinha.  
<img src="baguncinha.jpg" width="200">  
Voce so aceita. Entao explore.

# Incluindo e incluindo

Para que possamos incluir nosso modulo em um produto, precisamos encontrar algum arquivo de configuracao responsavel por manipular os artefatos finais daquele produto.  
No **meu** caso, que escolhi o produto 30, esse produto se encontra em **/device/generic/goldfish**.  

Os arquivos dele se encontram como se segue abaixo:  
```
.
├── Android.bp
├── Android.mk
├── AndroidProducts.mk
├── arm32-vendor.mk
├── arm64-vendor.mk
├── audio
├── camera
├── compatibility_matrix.xml
├── data
├── dhcp
├── display_settings_freeform.xml
├── emulator-info.txt
├── fingerprint
├── fstab.goldfish
├── fstab.ranchu
├── fstab.ranchu.arm
├── fstab.ranchu.arm.ex
├── fstab.ranchu.early
├── fstab.ranchu.early.arm
├── fstab.ranchu.ex
├── fstab.ranchu.initrd
├── fstab.ranchu.initrd.arm
├── fstab.ranchu.initrd.arm.ex
├── fstab.ranchu.initrd.ex
├── fstab.ranchu.initrd.noavb
├── fstab.ranchu.initrd.noavb.ex
├── fstab.ranchu.mips
├── fstab.ranchu.noavb
├── fstab.ranchu.noavb.ex
├── fvpbase
├── fvp.mk
├── gnss
├── init.goldfish.rc
├── init.goldfish.sh
├── init.ranchu-core.sh
├── init.ranchu-net.sh
├── init.ranchu.rc
├── input
├── input-mt
├── manifest.camera.xml
├── manifest.xml
├── MultiDisplayProvider
├── network
├── overlay
├── OWNERS
├── qemud
├── qemu-props
├── radio
├── rro_overlays
├── sdk_phone_x86_vendor.mk
├── sensors
├── sepolicy
├── soundtrigger
├── task_profiles.json
├── tnc
├── tools
├── ueventd.goldfish.rc
├── ueventd.ranchu.rc
├── vendor.mk
├── wifi
├── x86_64-vendor.mk
└── x86-vendor.mk
```

Cada arquivo serve pra alguma coisa diferente (claro, ne, Mr. obvio). (Iluminado).  

A gente se interessa aqui por algum arquivo que use a variavel de Makefile chamada **PRODUCT_PACKAGES**.  
Pelo que eu dei uma mexida, esse arquivo e o **vendor.mk**.

Para fazer funcionar entao temos que adicionar a essa variavel o nosso modulo.  
Podemos descer ate o final do arquivo, e escrever algo como:  
```
PRODUCT_PACKAGES += MinimalApp
```
Voce deve deixar uma linha em branco ao final do arquivo *.mk*. (eu sei por que?) (so deus sabe)

Feito isso, voce pode apertar **m** e recompilar o emulador todo.

Mas, porem, nao obstante, entretanto, ainda sim, nao sendo suficiente, vai dar erro.

Dentro do Android temos varias possiveis particoes, cada uma tendo um significado.  
Vai ler o manual e deixar o Sean orgulhoso: [Partitions](https://source.android.com/docs/core/architecture/bootloader/partitions), [product-interfaces](https://source.android.com/docs/core/architecture/bootloader/partitions/product-interfaces).  

Qual delas devemos usar? Quem diz qual?  

O proprio .bp, oras.

Para indicar qual particao desejamos usar, devemos dizer, ne?!

Assim sendo, podemos dizer usando as seguintes flags: 
```
system_ext_specific: true
```
```
device_specific: true
```
```
product_specific: true
```
```
soc_specific: true
```

Cada uma delas instala em uma particao especifica do device.  
E como eu sei todos esses comandos? Eu sou um dicionario de comandos do *blueprint*?  
Nao

Vai ler a documentacao:  
[soong_build.html](https://ci.android.com/builds/latest/branches/aosp-build-tools/targets/linux/view/soong_build.html)

Nos vamos usar a particao **/system_ext**. Porque sim. E a que eu trabaho no dia a dia e a que eu suponho corrigir qualquer asneira.

Para tanto, nosso .bp ficara como abaixo:  
```
android_app {
    name: "MinimalApp",
    srcs: ["**/*.kt"],
    platform_apis: true,
    certificate: "platform",
    privileged: true,
    system_ext_specific: true,
    optimize: {
        enabled: false,
    },
}
```

Caso voce queira seu artefato instalado na particao **/system**, voce pode excluir a linha referente ao **/system_ext**, excluir seu modulo la do arquivo **/device/generic/goldfish/vendor.mk** e adicionar ao arquivo **/build/target/product/base_system.mk**.  
Isso fara com que seu modulo fique em **/system**.  

Como eu sei? Eu li no proprio Android.  

Veja de exemplo o modulo **DownloadProvider**.  

Digite **mgrep DownloadProvider** e veja onde ele e incluido.

```
./platform_testing/build/tasks/tests/instrumentation_test_list.mk:56:    DownloadProviderTests \
./packages/providers/DownloadProvider/Android.bp:17:    name: "DownloadProvider",
./packages/providers/DownloadProvider/tests/Android.bp:17:    name: "DownloadProviderTests",
./packages/providers/DownloadProvider/tests/Android.bp:36:    instrumentation_for: "DownloadProvider",
./packages/providers/DownloadProvider/tests/permission/Android.bp:17:    name: "DownloadProviderPermissionTests",
./packages/providers/DownloadProvider/ui/CleanSpec.mk:47:$(call add-clean-step, rm -rf $(PRODUCT_OUT)/system/app/DownloadProviderUi)
./packages/providers/DownloadProvider/ui/Android.bp:17:    name: "DownloadProviderUi",
./packages/services/Car/car_product/build/car_base.mk:38:    DownloadProviderUi \
./build/make/target/product/handheld_system.mk:48:    DownloadProviderUi \
./build/make/target/product/base_system.mk:78:    DownloadProvider \
```

E sobre ler o manual.

# A decepcao

Para reconstruir o emulador com seu novo artefato, basta digitar: **m**.  

Ao ligar novamente, o que vera de novo? Nada.

**“Só se vê bem com o coração, o essencial é invisível aos olhos”**

O que criamos foi apenas uma casca.  

O minimo. Sem codigo. Sem icone. O nada.

Um coração escondido. Um homem de lata.

Para verificar que ele existe, basta digitar:  
```
adb shell pm list packages | grep com.venturus.minimal.app
```
Para ter certeza de onde, basta digitar:  
```
adb shell dumpsys package com.venturus.minimal.app | grep codePath
```
Voce deve ver algo como:
```
codePath=/system/system_ext/priv-app/MinimalApp
```
ou
```
codePath=/system_ext/priv-app/MinimalApp
```

<div style="display: flex; justify-content: center;">
    <img src="thatsall.gif" alt="![Thats all folks!](thatsall.gif)">
</div>