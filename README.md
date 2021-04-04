## Spring Boot com Maven BOM

# 1. Visão Geral
Neste tutorial rápido, vamos ver como o Maven, uma ferramenta baseada no conceito de Project Object Model (POM), pode fazer uso de um BOM ou “Bill Of Materials”.

Para obter mais detalhes sobre o Maven, você pode verificar nosso artigo Tutorial do Apache Maven.

# 2. Conceitos de gerenciamento de dependência
Para entender o que é um BOM e para que podemos usá-lo, primeiro precisamos aprender os conceitos básicos.

### 2.1. O que é Maven POM?
Maven POM é um arquivo XML que contém informações e configurações (sobre o projeto) que são usadas pelo Maven para importar dependências e construir o projeto.

### 2.2. O que é Maven BOM?
BOM significa lista de materiais. Um BOM é um tipo especial de POM usado para controlar as versões das dependências de um projeto e fornecer um local central para definir e atualizar essas versões.

O BOM fornece a flexibilidade de adicionar uma dependência ao nosso módulo sem nos preocupar com a versão da qual devemos depender.

### 2.3. Dependências transitivas
O Maven pode descobrir as bibliotecas que são necessárias para nossas próprias dependências em nosso pom.xml e incluí-as automaticamente. Não há limite para o número de níveis de dependência dos quais as bibliotecas são reunidas.

O conflito aqui surge quando 2 dependências se referem a diferentes versões de um artefato específico. Qual será incluído pelo Maven?

A resposta aqui é a “definição mais próxima”. Isso significa que a versão utilizada será a mais próxima do nosso projeto na árvore de dependências. Isso é chamado de mediação de dependência.

Vejamos o exemplo a seguir para esclarecer a mediação de dependência:

```
A -> B -> C -> D 1.4  and  A -> E -> D 1.0
```

Este exemplo mostra que o projeto A depende de B e E. B e E têm suas próprias dependências que encontram diferentes versões do artefato D. O artefato D 1.0 será usado na construção do projeto A porque o caminho através de E é mais curto.

Existem diferentes técnicas para determinar qual versão dos artefatos deve ser incluída:

- Sempre podemos garantir uma versão, declarando-a explicitamente no POM do nosso projeto. Por exemplo, para garantir que o D 1.4 seja usado, devemos adicioná-lo explicitamente como uma dependência no arquivo pom.xml;
- Podemos usar a seção Gerenciamento de dependências para controlar as versões dos artefatos, conforme explicaremos mais adiante neste artigo.

### 2.4. Gestão de Dependências
Simplificando, o Gerenciamento de Dependências é um mecanismo para centralizar as informações de dependência.

Quando temos um conjunto de projetos que herdam um pai comum, podemos colocar todas as informações de dependência em um arquivo POM compartilhado chamado BOM.

A seguir está um exemplo de como escrever um arquivo BOM:

```
<project ...>
	
    <modelVersion>4.0.0</modelVersion>
    <groupId>isaccanedo</groupId>
    <artifactId>isaccanedo-BOM</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>isaccanedo-BOM</name>
    <description>parent pom</description>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>test</groupId>
                <artifactId>a</artifactId>
                <version>1.2</version>
            </dependency>
            <dependency>
                <groupId>test</groupId>
                <artifactId>b</artifactId>
                <version>1.0</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>test</groupId>
                <artifactId>c</artifactId>
                <version>1.0</version>
                <scope>compile</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

Como podemos ver, o BOM é um arquivo POM normal com uma seção dependencyManagement onde podemos incluir todas as informações e versões de um artefato.

### 2.5. Usando o arquivo BOM
Existem 2 maneiras de usar o arquivo BOM anterior em nosso projeto e então estaremos prontos para declarar nossas dependências sem ter que nos preocupar com os números de versão.

Podemos herdar do pai:

```
<project ...>
    <modelVersion>4.0.0</modelVersion>
    <groupId>isaccanedo</groupId>
    <artifactId>Test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Test</name>
    <parent>
        <groupId>isaccanedo</groupId>
        <artifactId>isaccanedo-BOM</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
</project>
```

Como podemos ver, nosso projeto Test herda o isaccanedo-BOM.

Também podemos importar o BOM.

Em projetos maiores, a abordagem de herança não é eficiente porque o projeto pode herdar apenas um único pai. Importar é a alternativa, pois podemos importar quantas BOMs precisarmos.

Vamos ver como podemos importar um arquivo BOM para o nosso projeto POM:

```
<project ...>
    <modelVersion>4.0.0</modelVersion>
    <groupId>isaccanedo</groupId>
    <artifactId>Test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Test</name>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>isaccanedo</groupId>
                <artifactId>isaccanedo-BOM</artifactId>
                <version>0.0.1-SNAPSHOT</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 2.6. Sobrescrever dependência de BOM
A ordem de precedência da versão do artefato é:

1. A versão da declaração direta do artefato em nosso pom do projeto;
2. A versão do artefato no projeto pai;
3. A versão no pom importado, levando em consideração a ordem de importação dos arquivos;
4. mediação de dependência.

- Podemos sobrescrever a versão do artefato definindo explicitamente o artefato no pom do nosso projeto com a versão desejada;
- Se o mesmo artefato for definido com versões diferentes em 2 BOMs importados, a versão no arquivo de BOM que foi declarada primeiro vencerá.

# 3. Spring BOM
Podemos descobrir que uma biblioteca de terceiros, ou outro projeto Spring, puxa uma dependência transitiva para uma versão mais antiga. Se nos esquecermos de declarar explicitamente uma dependência direta, podem surgir problemas inesperados.

Para superar esses problemas, o Maven oferece suporte ao conceito de dependência de BOM.

Podemos importar o spring-framework-bom em nossa seção dependencyManagement para garantir que todas as dependências do Spring estejam na mesma versão:

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-framework-bom</artifactId>
            <version>4.3.8.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Não precisamos especificar o atributo version quando usamos os artefatos Spring como no exemplo a seguir:

```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
    </dependency>
<dependencies>
```

# 4. Conclusão

Neste artigo rápido, mostramos o conceito Maven Bill-Of-Material e como centralizar as informações e versões do artefato em um POM comum.

Simplificando, podemos herdar ou importá-lo para fazer uso dos benefícios do BOM.