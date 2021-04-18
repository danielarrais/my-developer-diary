---
title: 'Zipando arquivos com java'
tags: ["java", "zip"]
published: true
date: '2021-04-13'
---

# Zipar arquivos/pastas usando Java

Zipar arquivos usando o Java é uma tarefas simples, começando pelo fato de que ele conta com uma API nativa para tal, no
pacote `java.io`. E, para vermos essa simplicidade na prática, vamos começar esse artigo com a implementação do
zipamento de um arquivo, que será base para zipamento de múltiplos arquivos e de pastas.


## Zipar um arquivo

Considero bastante importante, antes de codificarmos a solução por mim proposta, entendermos algumas das classes que
vamos utilizar. Na minha solução precisamos principalmente destas classes: `java.io.FileOutputStream`
, `java.io.FileInputStream`, `java.util.zip.ZipOutputStream` e a `java.util.zip.ZipEntry`. A FileOutputStream é usada
para gravar bytes no disco e a FileInputStream para ler. Já a ZipOutputStream é uma FileOutputStream especializada em
criar e gravar arquivos zip's e a ZipEntry é utilizada para adionar arquivos dentro do ZIP, nela podemos adicionar
informações sobre o arquivo, como seu nome. No código abaixo podemos visualizar a solução proposta.

```java
// Busca o arquivo que queremos compactar
File file=new File("/home/daniel/teste_zip/gettyimages-521637760-170667a.jpg");

// Cria nosso "Winrar"
FileOutputStream fileOutputStream=new FileOutputStream("file.zip");
ZipOutputStream zipOutputStream=new ZipOutputStream(fileOutputStream);

// Cria o Stream que vai ler os bytes do nosso arquivo
FileInputStream fileInputStream=new FileInputStream(file);

// Cria uma entrada informando no contrutor o nome do arquivo
ZipEntry zipEntry=new ZipEntry(file.getName());

// Passa a entrada para nosso "WinRar"
zipOutputStream.putNextEntry(zipEntry);

// Adiciona o arquivo proprieamente dito ao nosso arquivo ZIP
byte[]bytes=new byte[1024];
int length;
while((length=fileInputStream.read(bytes))>=0){
    zipOutputStream.write(bytes,0,length);
}

// Fecha nosso Streams instânciados
fileInputStream.close();
zipOutputStream.close();
fileOutputStream.close();
```

O código é simples e consegue zipar um único arquivo, nele não há segredos:

1. primeiro criamos uma instância de File passando a URL do arquivo a ser compactado;
2. depois instanciamos nossos Streams, o FileOutputStream ****e ****ZipOutputStream;
3. criamos o FileInputStream que irá ler nosso arquivo e para isso passamos ele no construtor;
4. criamos uma instância ZipEntry passando o nome do nosso arquivo no construtor e depois passamos ela para a
   ZipOutputStream por meio do método putNextEntry;
5. depois lemos os bytes do arquivo e gravamos eles, compactados, usando nossa variável zipOutputStream;
6. por fim fechamos nossos streams instanciados.

Pronto! Zipamos um arquivo! Viu como é simples zipar arquivos usando somente classes nativas do Java? Mas não pararemos
por aqui, é possível a partir desse código criar soluções para compactar múltiplos arquivos e pastas! Sugiro que antes
de continuar a leitura, tente melhorar o código proposto e a partir do resultado crie suas próprias soluções. Deixo
apenas uma dica: na compactação das pastas a **recursividade** pode ser uma grande amiga... acho que entreguei o bolo
pronto rsrsrsrs.


## Zipar múltiplos arquivos

Antes de criarmos a solução vamos encapsular o código anterior em um método com a seguinte
assinatura: `void zipFile(File file, String zipName)`. A principal mudança fica a cargo do recebimento do arquivo e do
nome do zip como argumento. Além desse método vamos criar mais um, para adicionar um arquivo dentro do zip, com a
seguinte assinatura: `void addFileInZip(ZipOutputStream zipOutputStream, File file)`. Dentro deste método vamos colocar
a parte do código criado que adiciona o arquivo dentro dentro do zip e substituí-la no método anterior pela chamado do
método, como mostra o código a seguir.

```java
public static void zipFile(File file,String zipName)throws IOException{
  FileOutputStream fileOutputStream=new FileOutputStream(zipName);
  ZipOutputStream zipOutputStream=new ZipOutputStream(fileOutputStream);

  addFileInZip(zipOutputStream,file);

  zipOutputStream.close();
  fileOutputStream.close();
}

public static void addFileInZip(ZipOutputStream zipOutputStream,File file)throws IOException{
  FileInputStream fileInputStream=new FileInputStream(file);

  ZipEntry zipEntry=new ZipEntry(file.getName());
  zipOutputStream.putNextEntry(zipEntry);

  byte[]bytes=new byte[1024];
  int length;
  while((length=fileInputStream.read(bytes))>=0){
    zipOutputStream.write(bytes,0,length);
  }

  fileInputStream.close();
}
```

Agora que refatoramos nosso código e temos um método que adiciona um arquivo dentro do zip, podemos criar o novo método
que zipa vários arquivos. Nós vamos precisar receber nele uma lista de arquivos e o nome do arquivo zip, essa lista
deverá ser iterada e adicionada ao zip. O resultado é mostrado a seguir.


```java
public static void zipFiles(List<File> files, String zipName) throws IOException {
  FileOutputStream fileOutputStream = new FileOutputStream(zipName);
  ZipOutputStream zipOutputStream = new ZipOutputStream(fileOutputStream);

  for (File file : files) {
    addFileInZip(zipOutputStream, file);
  }

  zipOutputStream.close();
  fileOutputStream.close();
}
```


## Zipar pastas

A zipagem de uma pasta já é um pouco mais complexa - bem pouco mesmo - pois precisamos vasculhar ela em busca de todos
os arquivos e adicioná-los ao ZIP, e isso pode ser feito utilizando recursividade. Para isso vamos duplicar nosso
método `addFileInZip` e vamos adicionar mais um parâmetro do tipo String chamado `parentPath` em sua assinatura. A ideia
aqui é chamarmos o método `addFileInZip` passando a pasta e irmos chamando ele de forma recursiva para cada arquivo ou
pasta encontrada. Vendo a ideia já nos deparamos com um problema: teremos pastas e arquivos, então nosso método deverá
está preparado para processar pasta ou arquivo. A seguir a solução proposta para facilitar o entendimento.

```java
private static void addFileInZip(File file, ZipOutputStream zipOutputStream, String parentPath) throws IOException {
  if (file == null || file.isDirectory() && file.listFiles() == null) {
    return;
  }

  if (file.isDirectory()) {
     for (File childFile : file.listFiles()) {
        addFileInZip(childFile, zipOutputStream, parentPath + "/" + file.getName());
     }
  } else {
     FileInputStream fileInputStream = new FileInputStream(file);
   
     ZipEntry zipEntry = new ZipEntry(parentPath + "/" + file.getName());
     zipOutputStream.putNextEntry(zipEntry);
   
     byte[] bytes = new byte[1048576];
     int length;
     while ((length = fileInputStream.read(bytes)) >= 0) {
        zipOutputStream.write(bytes, 0, length);
     }

    fileInputStream.close();
  }
}
```

Para lidar se o File passado é um arquivo ou uma pasta utilizei o método `isDirectory()`. Quando ele retorna `true`
itero a lista de arquivos da pasta - obtida por meio do método `listFiles()`, chamando o método de forma recursiva,
passando cada um dos arquivos para ele e concatenando o nome do arquivo/pasta com a variável parentPath. Assim a medida
que o método vai explorando a pasta ele vai recebendo o caminho correto da pasta. Caso o resultado dê `falso` ele faz o
processamento normal que já fizemos, com a diferença na concatenação do path da pasta pai com o nome do arquivo.

E como você deve ter notado, agora o código do primeiro método `addFileInZip` está duplicado no segundo criado. Para
resolvermos isso você pode substituir todo o código do primeiro por uma chamada do segundo método, passando no terceiro
paramêtro uma String vazia. Assim deixamos nosso código sem duplicidade e mais simples de manter.

Com essa refatoração vemos que não precisamos criar um método exclusivo para compactar pasta, pois o `zipFile` já vai
conseguir realizar tal tarefa. Porém você pode para ficar mais claro renomêá-lo para `zipFileOrFolder` por exemplo, ou
então criar um método chamado `zipFolder` que chama o `zipFile`.

O resultado final pode ser visualizado no gist abaixo:

<script src="https://gist.github.com/danielarrais/bbdb631d2dcc41baf18cc6f914e2ccb6.js"></script>


