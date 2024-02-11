# Como criar um bom sistema de Log

_Programar é a arte de tornar a lógica bela o suficiente para todos apreciarem._

Você já se encontrou perdido em um labirinto de logs, tentando decifrar mensagens enigmáticas como se estivesse decifrando hieróglifos egípcios? Bem-vindo ao clube! Mas calma, respira fundo e relaxa, porque hoje estamos prestes a desvendar esse enigma juntos.

No meu próximo mergulho aqui no universo da programação para jogos, vamos explorar algo que muitas vezes é negligenciado, mas que é absolutamente crucial para a manutenção e o aprimoramento de qualquer jogo: os logs! Sim, aqueles registros misteriosos que são como a caixa preta dos nossos programas.

Então, prepare-se para uma jornada cheia de diversão e aprendizado, enquanto descobrimos como criar um formato de log que seja tão claro quanto o céu azul e tão útil quanto um canivete suíço em um acampamento. Ao longo deste post você verá alguns imports com **com.vgames.survivalreckoning**, bem, estou usando o sistema ensinado aqui em meu projeto pessoal, mas como gostei bastante do formato, decidi compartilhá-lo aqui, mas não se preocupe, a única coisa que você mudará em sua implementação será o nome do pacote. Vamos lá!

## Parte 1 - Configurações

Antes de mergulharmos no núcleo desse microsistema que injetaremos em nossa aplicação, criaremos as classes de configuração, bem... não exatamente classes. Como nosso módulo de Log será feito em Java, tiraremos proveito disso e utilizaremos **annotations**. Começaremos então com a seguinte annotation:

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)

@Target({ElementType.TYPE})
public @interface LogAlias {
    String value();
}
```
Essa annotation será usada em Runtime para darmos apelidos as classes que utilizarão nosso Logger.

Não se assuste com isso, é um trecho normal de Java, caso você já esteja habituado com isso, pode ir em frente e pular essa [próxima explicação](#Outras-classes-de-configuração), caso contrário, recomendo que leia o que vem a seguir.

# Annotations em Java

As annotations em Java são uma forma de especificar metadados de qualquer coisa "palpável" da linguagem, como classes, records, métodos, até mesmo atributos. Por exemplo, quando você anota uma classe com **@Override** você especificando que um método deverá ser sobreposto por uma classe filha. Outro exemplo é por exemplo quando você faz uso potencialmente inseguro de tipos parametrizados (generics) e precisa anotar um método ou classe com **@SupressWarnings("unchecked")**. Seguem exemplos abaixo:

```java
import com.vgames.survivalreckoning.service.audio.Sound;

// Interface que objetos audiveis devem implementar.
public interface Audible {
    void play(Sound sound);
}
```

```java
public abstract class Animal implements Audible {

    public abstract void emitSound();
}
```

```java
public class Dog extends Animal {

    // Observe a presença da annotation @Override no metodo.
    @Override
    public void emitSound() {

        // Aqui obtemos o asset de som chamado DogSound001. A classe SoundService retorna um objeto do tipo Sound.
        play(SoundService.get("DogSound001"));
    }

    @Override
    public void play(Sound sound) {
        
        // Aqui obtemos o serviço global de audio, em seguida a classe que reproduz efeitos sonoros e reproduzimos o som que o cachorro deve emitir. Para este exemplo, temos apenas um unico som, mas ja serve.
        Engine.fromService(AudioAPI.class).getSoundEffectPlayer().play(sound);
    }
}
```

Agora voltando no contexto da nossa annotation, observe novamente o código com comentários.

```java
// Imports necessarios para implementar uma annotation.
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

// Aqui especificamos o tipo de retenção da annotation.
// Existem 3 tipos: Runtime, Class e Source.

/*
Na retenção do tipo Source, a annotation será apenas um elemento visual no 
código. Não podendo ser usado em tempo de compilação ou execução.

Na retenção do tipo Class, a annotation estará disponivel durante o tempo de compilação.
Isso significa que poderemos utilizá-la em pre-processamento do código, mas não 
em tempo de execução.

Na retenção do tipo Runtime, a annotation estará disponivel durante todo o tempo 
de vida da aplicação, e poderemos acessá-la via Reflection.

Guarde bem este ultimo trecho que fala sobre a retenção do tipo Runtime, 
usaremos ele mais tarde.
*/
@Retention(RetentionPolicy.RUNTIME)

// A annotation @Target indica onde nossa annotation pode ser usada, podemos 
//passar uma ou mais especificações, no nosso caso usaremos apenas ElementType.
//TYPE (nesse caso, TYPE representa classes).

// Caso quisessemos que a annotation pudesse ser usada em métodos, como é o caso
//de @Override, poderiamos adicionar ElementType.METHOD na lista de especificações.

// Você pode encontrar todos os tipos de especificações aqui: 
//https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/ElementType.html
@Target({ElementType.TYPE})

// A forma de declarar uma annotation é essa mesmo, com @interface.
public @interface LogAlias {

    // Os atributos de uma annotation são declarados como métodos, 
    //então nossa annotation tem um atributo do tipo String chamado value.
    String value();
}
```

# Outras classes de configuração

Agora, criaremos as outras annotations que utilizaremos para configurar nosso sistema.

```java
import com.vgames.survivalreckoning.log.LogLevel;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface LogInfo {
    LogLevel level() default LogLevel.DEBUG;
    boolean verbose() default true;
}
```
Os atributos level e verbose definem alguns aspectos importantes, o primeiro define qual a importância daquele Logger específico, por exemplo, se nossa aplicação está em estado de erro crítico, não faz sentido emitir mensagens com informações básicas pois elas dificultariam a visualização do que realmente importa naquele momento. Então vamos em frente, para usar nosso atributo **level** precisamos do Enum **LogLevel**:

```java
public enum LogLevel {
    CRITICAL, ERROR, WARN, TRACE, INFO, DEBUG
}
```
O segundo atributo, **verbose** indica se queremos um log completo, ou apenas uma visão resumida do mesmo, por exemplo, exceções relacionadas a emissão errada de um efeito visual do jogo, como uma partícula de folha, não deve ser detalhadamente explicada, enquanto uma exceção relacionada a falha na abertura de um arquivo de salvamento deve ter um espaço dedicado a ela. A última classe reterá a configuração de depuração, ou seja, se um Logger conter essa annotation, seus logs não serão emitidos na versão de testes da aplicação:

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
public @interface NotDebugLog {
}
```

A última configuração é relacionada a geração de arquivos em pontos criticos da aplicação:

```java
// GenerateCriticalFile.java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface GenerateCriticalFile {
}

```

Certo, terminamos tudo relacionado a configuração, podemos partir para a classe Logger de fato, vamos em frente:

```java
// Logger.java

public abstract class Logger {

    private static final boolean globalDebugDefinition = true;

    private final String name;
    private final DateFormat dateFormat;
    private LogLevel logLevel;
    private boolean isVerboseException;
    private boolean isDebugging;
    private boolean generateCriticalFile;
}
```

Primeiro definimos alguns atributos que usaremos na formatação de nosso logs, juntamente com as propriedades que nossas annotations alterarão.

```java
// Restante do codigo...

private boolean isDebugging;


protected Logger() {
    if(getClass().isAnnotationPresent(LogAlias.class)) {
        String alias = getClass().getDeclaredAnnotation(LogAlias.class).value();
        this.name = alias.isEmpty() ? getClass().getSimpleName() : alias;
    } else {
        this.name = getClass().getSimpleName();
    }
    this.dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    defineProperties(getClass());
}
```

Nosso construtor já define algumas coisas, primeiro, verificamos se a annotation **@LogAlias** está presente em nosso Logger usando o método **getClass**, quando usamos isso, a classe retornada será a classe que herda de Logger pois a linguagem Java da prioridade as classes filhas. Em seguida, caso a annotation **@LogAlias** esteja presente, obtemos o valor do atributo **value** contido nela, por garantia, checamos se o apelido passado não é uma String vazia, caso seja, o nome do Logger será o nome da própria classe, caso contrário, será o apelido definido. EM seguida definimos o formato da data de geração dos logs e chamamos o método **defineProperties** para terminar o trabalho. Abaixo segue um exemplo de saída de um log com e sem apelidos:


Sem LogAlias:

```java
public class Application extends Logger {

    public Application() {
        warn("Application has been initialized.");
    }
}

// Saída quando tivermos a classe completa:
// [2024-02-10 21:34:23] WARN      Application: Application has been initialized
```

Com LogAlias:

```java
@LogAlias("Survival Reckoning")
public class Application extends Logger {

    public Application() {
        warn("Application has been initialized.");
    }
}

// Saída quando tivermos a classe completa:
// [2024-02-10 21:34:23] WARN      Survival Reckoning: Application has been initialized
```

Em seguida, criamos o método **defineProperties**:

```java
// Restante do codigo...
    this.dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    defineProperties(getClass());
}


private <T> void defineProperties(Class<T> klass) {
    if (klass.isAnnotationPresent(LogInfo.class)) {
        LogInfo info = klass.getDeclaredAnnotation(LogInfo.class);
        this.logLevel = info.level();
        this.isVerboseException = info.verbose();
    } else {
        this.logLevel = LogLevel.DEBUG;
        this.isVerboseException = true;
    }

    if(klass.isAnnotationPresent(GenerateCriticalFile.class)) {
        this.generateCriticalFile = true;
    } else {
        this.generateCriticalFile = false;
    }

    this.isDebugging = !klass.isAnnotationPresent(NotDebugLog.class) && globalDebugDefinition;
}
```

Se você entendeu como as annotations funciona, terá facilidade em entender este trecho, apenas verificamos a presença da annotation **@LogInfo** na classe, caso ela exista, obtemos os valores de seus atributos, repare que a checagem de campos nulos não é necessária pois definimos valores padrão (default) em nossas annotations para o caso de um ou mais deles não serem passados. Caso a annotation não seja encontrada, definimos o level minimo do log para **DEBUG** e a formatação verbosa como **true**.

Agora vamos criar um método para obter o tempo atual formatado:

```java
// Restante do codigo...

    this.isDebugging = !klass.isAnnotationPresent(NotDebugLog.class) && globalDebugDefinition;
}


private String getFormattedTimestamp() {
    return dateFormat.format(new Date());
}
```

Precisamos também de algo para formatar as mensagens em um formato amigável:

```java
// Restante do codigo

private String getFormattedTimestamp() {
    return dateFormat.format(new Date());
}


private String formatLogMessage(String level, String message, String color) {
    int maxSpacesLength = Arrays.stream(values()).mapToInt(_level -> _level.name().length()).max().orElse(0);

    int currentSpacesLength = level.length();

    String formattedName = " ".repeat(1 + (maxSpacesLength - currentSpacesLength)) + BOLD + name + RESET;

    return String.format("[%s] %s %s: %s", getFormattedTimestamp(), color + level + RESET, formattedName, message);
}
```

Este método necessita de algumas constantes definidas em outra classe, nada de especial além de literalmente formatar as mensagens de log, vamos criar a classe com as constantes agora:

```java
// LogColor.java
class LogColor {
    static final String RESET = "\u001B[0m";           // Default properties
    static final String BOLD = "\u001B[1m";            // Bold
    static final String COLOR_INFO = "\u001B[36m";     // Cyan
    static final String COLOR_TRACE = "\u001B[37m";    // White
    static final String COLOR_WARN = "\u001B[33m";     // Yellow
    static final String COLOR_ERROR = "\u001B[31m";    // Red
    static final String COLOR_CRITICAL = "\u001B[35m"; // Purple
}
```

Esta classe usa os código de escape ANSI para colorir as mensagens de log. Não se esqueça de usar estes dois imports de forma estática:

```java
import static com.vgames.survivalreckoning.log.LogColor.*;
import static com.vgames.survivalreckoning.log.LogLevel.*;
// Adicione no inicio de Logger.java
```

O próximo método é dedicado a formatar exceções, pois como definimos antes, exceções podem ser verbosas ou não:

```java
// Restante do codigo

    return String.format("[%s] %s %s: %s", getFormattedTimestamp(), color + level + RESET, formattedName, message);
}


private String formatExceptionMessage(Throwable throwable) {
    if (isVerboseException) {
        StringBuilder exceptionMessage = new StringBuilder();
        exceptionMessage.append(throwable.getClass().getName()).append(": ").append(throwable.getMessage()).append("\n");
        for (StackTraceElement element : throwable.getStackTrace()) {
            exceptionMessage.append("\t").append(element.toString()).append("\n");
        }
        return exceptionMessage.toString();
    } else {
        return throwable.getClass().getSimpleName() + (throwable.getMessage() != null ? ": " : "") + throwable.getMessage();
    }
}
```
A seguir, definiremos um método que usará as formatações e a classe com as sequencias de escape ANSI para _printar_ as mensagens:

```java
// Restante do codigo

        return throwable.getClass().getSimpleName() + (throwable.getMessage() != null ? ": " : "") + throwable.getMessage();
    }
}


private void printMessage(LogLevel level, String message, String color) {
    if (level.ordinal() <= this.logLevel.ordinal()) {
        System.out.println(formatLogMessage(level.name(), message, color));
    }
}
```

Repare que checamos o nivel do log para saber se podemos imprimir a mensagem. Os próximos métodos usarão o método **printMessage** para formatar adequadamente suas mensagens:

```java
// Restante do codigo 

private void printMessage(LogLevel level, String message, String color) {
    if (level.ordinal() <= this.logLevel.ordinal()) {
        System.out.println(formatLogMessage(level.name(), message, color));
    }
}

protected void debug(String message) {
    if (isDebugging) {
        printMessage(DEBUG, message, COLOR_INFO);
    }
}

protected void info(String message) {
    if (isDebugging) {
        printMessage(INFO, message, COLOR_INFO);
    }
}

protected void trace(String message) {
    if (isDebugging) {
        printMessage(TRACE, message, COLOR_TRACE);
    }
}

protected void warn(String message) {
    printMessage(WARN, message, COLOR_WARN);
}

protected void error(String message) {
    printMessage(ERROR, message, COLOR_ERROR);
}

protected void critical(String message) {
    printMessage(CRITICAL, message, COLOR_CRITICAL);
}

protected void error(String message, Throwable throwable) {
    printMessage(ERROR, message, COLOR_ERROR);
    logException(throwable);
}

protected void critical(String message, RuntimeException exception) {
    printMessage(CRITICAL, message, COLOR_CRITICAL);

    if(this.generateCriticalFile) {
        generateCriticalFile(exception);
    }

    throwException(exception);
}
```

Nosso último passo é adicionar o metodo para gerar um arquivo de log nos pontos criticos, ele conterá o horário da exceção, o erro em si, a mensagem do erro, e a stack trace gerada:

```java
// Restante do codigo...

    if(this.generateCriticalFile) {
        generateCriticalFile(exception);
    }

    throwException(exception);
}


private void generateCriticalFile(RuntimeException exception, String message) {
    if (generateCriticalFile) {
        try (PrintWriter writer = new PrintWriter(new FileWriter(getFormattedTimestamp().replaceAll(" ", "_").replaceAll(":", "-") + ".log"))) {
            writer.println("Critical Error:");
            writer.println("Timestamp: " + getFormattedTimestamp());
            writer.println("Message: " + message);
            writer.println("Stack Trace:");
            exception.printStackTrace(writer);
            writer.println("\nAdditional Details:");
            writer.println("Exception Type: " + exception.getClass().getName());
            writer.println("Exception Message: " + exception.getMessage());
            writer.println("Exception Cause: " + (exception.getCause() != null ? exception.getCause().toString() : "N/A"));
            writer.println("Exception Source: " + getExceptionSource(exception));
        } catch (IOException e) {
            error(message, e);
        }
    }
}

private String getExceptionSource(Exception exception) {
    StackTraceElement[] elements = exception.getStackTrace();
    if (elements.length > 0) {
        return elements[0].toString();
    }
    return "Unknown";
}
```

Finalmente acabamos! Podemos testar nosso sistema, o exemplo que faremos adiciona mais algumas propriedades a classe **Application** que criamos anteriormente:

```java
@LogAlias("Survival Reckoning")
@GenerateCriticalFile
public class Application extends Logger {

    public Application() {
        if(init()) {
            info("Application has been initialized.");
        } else {
            critical("Failed to initialize services.", new IllegalStateException());
        }
    }

    private boolean init() {
        return true;
    }
}
```

Ao executar este codigo (tendo um método main e uma instancia de Application), podemos observar a seguinte saída:

```Bash
[2024-02-10 22:26:53] INFO      Survival Reckoning: Application has been initialized.
```

Vamos alterar o valor retornado por **init** para false:

```java
@LogAlias("Survival Reckoning")
@GenerateCriticalFile
public class Application extends Logger {

    public Application() {
        if(init()) {
            info("Application has been initialized.");
        } else {
            critical("Failed to initialize services.", new IllegalStateException());
        }
    }

    private boolean init() {
        return true;
    }
}
```

Desta vez a saída será a seguinte, não repare nos nomes de classes, isso é a saída da minha aplicação real quando os serviços não puderem ser inicializados corretamente:

```Bash
[2024-02-10 22:27:57] CRITICAL  Application: Failed to initialize services:  AudioAPI
Exception in thread "main" java.lang.IllegalStateException
	at com.vgames.survivalreckoning.engine.Engine.initializeServices(Engine.java:64)
	at com.vgames.survivalreckoning.engine.Engine.init(Engine.java:30)
	at com.vgames.survivalreckoning.application.Application.init(Application.java:15)
	at com.vgames.survivalreckoning.Main.main(Main.java:7)
```

Além disso, observe o arquivo gerado no diretório raiz do seu projeto (será o mesmo diretório configurado pelo seu classpath ao executar a JVM). No meu caso, ele possui o seguinte conteudo:

```java
Critical Error:
Timestamp: 2024-02-10 22:31:45
Message: Failed to initialize services:  AudioAPI
Stack Trace:
java.lang.IllegalStateException
	at com.vgames.survivalreckoning.engine.Engine.initializeServices(Engine.java:64)
	at com.vgames.survivalreckoning.engine.Engine.init(Engine.java:30)
	at com.vgames.survivalreckoning.application.Application.init(Application.java:15)
	at com.vgames.survivalreckoning.Main.main(Main.java:7)

Additional Details:
Exception Type: java.lang.IllegalStateException
Exception Message: null
Exception Cause: N/A
Exception Source: com.vgames.survivalreckoning.engine.Engine.initializeServices(Engine.java:64)
```

Pronto! Agora conseguimos também salvar as exceções críticas lançadas pela aplicação. 

## Conclusão

Em resumo, criar um bom sistema de log é essencial para o desenvolvimento e manutenção de qualquer aplicação, especialmente em jogos, onde o rastreamento de erros e o monitoramento do desempenho são cruciais para a experiência do usuário.

Ao longo deste guia, exploramos como configurar e implementar um sistema de log eficaz em Java, utilizando annotations para personalizar o comportamento do logger. Começamos definindo annotations como @LogAlias, @LogInfo e @NotDebugLog, que nos permitem especificar apelidos para classes, níveis de log e se os logs devem ser incluídos em versões de depuração.

Em seguida, desenvolvemos a classe Logger, que fornece métodos para registrar mensagens de log em diferentes níveis de gravidade, como debug, info, warn, error e critical. Além disso, implementamos a geração de arquivos de log em pontos críticos da aplicação, fornecendo detalhes adicionais sobre exceções lançadas.

Por fim, testamos nosso sistema com uma aplicação de exemplo, demonstrando como as mensagens de log são formatadas e exibidas no console, e como os arquivos de log são gerados em caso de erros críticos.

Em suma, ao seguir as práticas recomendadas e personalizar seu sistema de log de acordo com as necessidades específicas da sua aplicação, você estará melhor equipado para detectar, diagnosticar e corrigir problemas, garantindo uma experiência de usuário mais estável e satisfatória.

### Faça bom proveito do conhecimento adquirido aqui, e até a próxima.