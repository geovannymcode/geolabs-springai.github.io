# **Chat Model**

## **Paso 1: Configuración de la Clave API de OpenAI**

Para que nuestra aplicación pueda acceder a los modelos de OpenAI, necesitamos una clave API válida de OpenAI. Esta clave permite autenticar las solicitudes y garantiza que la aplicación tenga permiso para utilizar los modelos de lenguaje. Asegúrate de tener una cuenta en OpenAI y de obtener una clave API desde su sitio web.

Una vez tengas la clave, debes configurarla en el archivo `application.yml` para que esté disponible en el proyecto. La configuración debería verse así:

```yaml title="Application.yml" linenums="1"
spring:
  ai:
    openai:
      api-key: `SPRING_AI_OPENAI_API_KEY`
```

- **Nota**: Reemplaza `SPRING_AI_OPENAI_API_KEY` con tu clave API real de OpenAI.

Esta configuración permitirá que nuestra aplicación Spring Boot acceda a la clave a través de la anotación `@Value` en la clase de configuración `OpenAIConfig`.

## **Paso 2: Clase de Configuración `OpenAIConfig`**

Para crear una instancia de **OpenAiChatModel** (la clase que interactúa con los modelos de OpenAI), necesitamos una clase de configuración que inyecte la clave API y configure los parámetros básicos para las solicitudes de generación de texto. Esta clase es `OpenAIConfig`.

```java title="OpenAIConfig.java" linenums="1"
@Configuration
public class OpenAIConfig {

    @Value("${spring.ai.openai.api-key}")
    private String API_KEY;

    @Bean
    public OpenAiChatModel openAiChatModel() {
        final OpenAiApi openAiApi = new OpenAiApi(API_KEY);

        return new OpenAiChatModel(openAiApi, OpenAiChatOptions.builder()
                .withModel("gpt-3.5-turbo")
                .withTemperature(0.8)
                .withMaxTokens(200)
                .build()
        );
    }
}
```

- **Línea 1**: `@Configuration` Anota la clase para que Spring la reconozca como una clase de configuración. Esto significa que Spring gestionará los beans definidos dentro de esta clase.

- **Línea 4**: `@Value("${spring.ai.openai.api-key}")` Inyecta el valor de la clave API de OpenAI desde el archivo application.yml. Esta clave es necesaria para autenticar las solicitudes al servicio de OpenAI.

- **Línea 8**: `Método openAiChatModel()` Este método define un bean de tipo `OpenAiChatModel`, que es el cliente que utilizaremos para enviar solicitudes a OpenAI.

- **Línea 9**: `OpenAiApi` Se inicializa con la clave API, permitiendo la comunicación segura con OpenAI.
- **Línea 11**: `OpenAiChatModel` Esta instancia se configura con `OpenAiChatOptions` para definir parámetros específicos como:
- **Línea 12**: `withModel("gpt-3.5-turbo")` Especifica el modelo que utilizaremos, en este caso, gpt-3.5-turbo.
- **Línea 13**: `withTemperature(0.8)` Controla la creatividad de las respuestas generadas. Valores más bajos hacen las respuestas más conservadoras, mientras que valores más altos (hasta 1) hacen las respuestas más creativas.
- **Línea 14**: `withMaxTokens(200)` Define el límite de tokens (unidades de texto) en la respuesta. Esto evita que las respuestas sean demasiado largas y controla el costo de las solicitudes.
- **Línea 15**: `build()`: Este método se utiliza para finalizar la construcción de OpenAiChatOptions y devuelve una instancia configurada de estas opciones. Sin build(), la configuración del cliente quedaría incompleta y lanzaría un error.

Este bean estará disponible para ser inyectado en otras clases, como el controlador `ChatController`, permitiéndonos realizar llamadas a la API de OpenAI.

## **Paso 3: Clase `ResponseDTO`**

La clase `ResponseDTO` es un **Data Transfer Object (DTO)** que permite estructurar las respuestas enviadas desde los endpoints en el controlador. Este patrón es útil para mantener consistencia en las respuestas, asegurando que cada respuesta contenga un estado, un mensaje y los datos específicos.

```java title="ChatController.java" linenums="1"
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Setter
public class ResponseDTO<T> {
    private int status;
    private String message;
    private T data;
}
```

### **Anotaciones y Propiedades**

- `@AllArgsConstructor`: Genera un constructor que acepta valores para todos los atributos (`status`, `message`, y `data`), lo cual facilita la creación de instancias con todos los campos inicializados.

- `@NoArgsConstructor`: Genera un constructor sin argumentos, permitiendo la creación de instancias vacías de `ResponseDTO`.

- `@Getter` y `@Setter`: Estas anotaciones de Lombok generan automáticamente los métodos `get` y `set` para cada propiedad, simplificando el acceso y modificación de los atributos sin necesidad de escribirlos manualmente.

### **Propiedades**

- `status` (`int`): Representa el código de estado de la respuesta (por ejemplo, `200` para éxito, `400` para errores de cliente, etc.). Esto facilita la interpretación del estado de la solicitud en el cliente.

- `message` (`String`): Proporciona un mensaje descriptivo sobre el resultado de la solicitud. Este mensaje puede ser "success", "error", u otra descripción, y ayuda al cliente a entender mejor el contexto de la respuesta.

- `data` (`T`): Representa los datos específicos devueltos por el endpoint. Esta es una propiedad genérica (`<T>`), lo que significa que puede contener cualquier tipo de objeto. Esto permite que `ResponseDTO` sea reutilizable en distintos endpoints, adaptándose a las necesidades de cada uno.

## **Paso 4:Controlador `ChatController`**

El controlador `ChatController` expone dos endpoints para interactuar con la IA de OpenAI. A través de estos endpoints, los usuarios pueden enviar mensajes y obtener respuestas generadas por el modelo de IA. Este controlador permite:

- Generación de texto simple en base a un mensaje.
- Generación de conversaciones manteniendo un historial de mensajes, para que el modelo pueda tener contexto en las respuestas.

```java title="ChatController.java" linenums="1"
@RestController
@RequestMapping("/chats")
@CrossOrigin(origins = "*")
@RequiredArgsConstructor
public class ChatController {

    private final OpenAiChatModel openAiChatModel;
    private final ChatHistory chatHistory;

    @GetMapping("/generate")
    public ResponseEntity<ResponseDTO<String>> generateText(@RequestParam String message) {
        ChatResponse chatResponse = openAiChatModel.call(new Prompt(message));
        String result = chatResponse.getResult().getOutput().getContent();

        return ResponseEntity.ok(new ResponseDTO<>(200, "success", result));
    }

    @GetMapping("/generateConversation")
    public ResponseEntity<ResponseDTO<String>> generateConversation(@RequestParam String message){
        chatHistory.addMessage("1", new UserMessage(message));

        ChatResponse chatResponse = openAiChatModel.call(new Prompt(chatHistory.getAll("1")));
        String result = chatResponse.getResult().getOutput().getContent();

        return ResponseEntity.ok(new ResponseDTO<>(200, "success", result));
    }
}
```

- **Línea 1**: `@RestController` Indica que esta clase es un controlador REST. Spring gestionará automáticamente las solicitudes HTTP y las respuestas en formato JSON.

- **Línea 2**: `@RequestMapping("/chats")` Define el prefijo de la URL para todos los endpoints de este controlador (`/chats`).

- **Línea 3**: `@CrossOrigin(origins = "*")` Permite solicitudes CORS desde cualquier origen, lo cual es útil para desarrollo. En producción, puedes restringirlo a un dominio específico.

- **Línea 4**: `@RequiredArgsConstructor` Genera un constructor para inyectar los componentes `openAiChatModel` y `chatHistory` (gestión del historial de la conversación), facilitando la inyección de dependencias.

### **Métodos en el `ChatController`**

1. **Línea 10**: `generateText (@GetMapping("/generate"))`
       - **Objetivo**: Generar una respuesta de texto simple en base a un mensaje proporcionado por el usuario.
       - **Parámetro**: `message`, que contiene el texto enviado por el usuario.
       - **Lógica**:
          - Crea una instancia de `Prompt` con el mensaje del usuario.
          - Llama al método `call` de `openAiChatModel`, que envía el `Prompt` a OpenAI.
          - Obtiene la respuesta generada por el modelo (`chatResponse`) y extrae el contenido de texto (`result`).
       - **Respuesta**: Devuelve un `ResponseDTO` con el estado y el texto generado.

2. **Línea 18**: `generateConversation (@GetMapping("/generateConversation"))`
       - **Objetivo**: Generar una respuesta basada en una conversación con contexto.
       - **Parámetro**: message, que representa el mensaje actual del usuario.
       - **Lógica**:
          - Agrega el mensaje al historial de conversación (chatHistory) usando un identificador de conversación (`"1"`).
          - Envía todo el historial de mensajes al modelo, en lugar de solo el mensaje actual. Esto permite que el modelo de IA genere respuestas considerando el contexto.
          - Obtiene y devuelve la respuesta generada, al igual que en el método generateText.
       - **Respuesta**: Devuelve un `ResponseDTO` con el estado y la respuesta generada por el modelo de IA.

## **Paso 5:Clase `ChatHistory`**

La clase `ChatHistory` es un componente de Spring (`@Component`) que almacena el historial de mensajes para cada conversación y permite realizar un seguimiento de los mensajes agregados. Esto permite que el modelo de IA tenga contexto en las respuestas, lo cual es útil para conversaciones continuas en las que cada mensaje depende de los anteriores.

```java title="ChatController.java" linenums="1"
@Component
public class ChatHistory {

    private static final Logger logger = LoggerFactory.getLogger(ChatHistory.class);

    private final Map<String, List<Message>> chatHistoryLog;
    private final Map<String, List<Message>> messageAggregations;

    public ChatHistory() {
        this.chatHistoryLog = new ConcurrentHashMap<>();
        this.messageAggregations = new ConcurrentHashMap<>();
    }

    public void addMessage(String chatId, Message message) {
        String groupId = toGroupId(chatId, message);

        this.messageAggregations.computeIfAbsent(groupId, key -> new ArrayList<>()).add(message);

        if (this.messageAggregations.size() > 1) {
            logger.warn("Multiple active sessions: {}", this.messageAggregations.keySet());
        }

        String finishReason = getProperty(message, "finishReason");
        if ("STOP".equalsIgnoreCase(finishReason) || message.getMessageType() == MessageType.USER) {
            this.finalizeMessageGroup(chatId, groupId);
        }
    }

    private String toGroupId(String chatId, Message message) {
        String messageId = getProperty(message, "id");
        return chatId + ":" + messageId;
    }

    private String getProperty(Message message, String key) {
        return message.getMetadata().getOrDefault(key, "").toString();
    }

    private void finalizeMessageGroup(String chatId, String groupId) {
        List<Message> sessionMessages = this.messageAggregations.remove(groupId);

        if (sessionMessages != null) {
            if (sessionMessages.size() == 1) {
                this.commitToHistoryLog(chatId, sessionMessages.get(0));
            } else {
                String aggregatedContent = sessionMessages.stream()
                        .map(Message::getContent)
                        .filter(Objects::nonNull)
                        .collect(Collectors.joining());
                this.commitToHistoryLog(chatId, new AssistantMessage(aggregatedContent));
            }
        } else {
            logger.warn("No active session for groupId: {}", groupId);
        }
    }

    private void commitToHistoryLog(String chatId, Message message) {
        this.chatHistoryLog.computeIfAbsent(chatId, key -> new ArrayList<>()).add(message);
    }

    public List<Message> getAll(String chatId) {
        return this.chatHistoryLog.getOrDefault(chatId, List.of());
    }
}
```

### **Componentes de `ChatHistory`**

- **Línea 1**: `@Component` Indica que esta clase es un componente de Spring, lo que permite que Spring la gestione como un bean y la inyecte en otras clases.

- **Línea 1**: Logger: Se utiliza Logger para registrar advertencias, especialmente si se detectan múltiples sesiones activas o si no hay una sesión activa para un grupo de mensajes específico.

- **Estructuras de Datos**:
      - **Línea 1**: chatHistoryLog (Map<String, List<Message>>): Este mapa almacena el historial completo de cada conversación, agrupado por chatId. Cada chatId tiene una lista de mensajes que representa toda la conversación.
      - **Línea 1**: messageAggregations (Map<String, List<Message>>): Almacena mensajes agrupados temporalmente por sesión antes de que se finalicen y se registren en chatHistoryLog. Esta estructura permite manejar múltiples mensajes en una sesión, útil para conversaciones donde la IA responde en varias partes.

### **Métodos en ChatHistory**

- **Línea 1**: addMessage(String chatId, Message message)
      - **Propósito**: Agrega un mensaje a la sesión actual o crea una nueva si es necesario.
      - **Parámetros**:
          - **chatId**: Identificador de la conversación.
          - **message**: El mensaje que se quiere agregar.
      - **Lógica**:
          - Genera un `groupId` único para la sesión actual usando `chatId` y `message`.
          - Agrega el mensaje a messageAggregations, y si hay varias sesiones activas, registra una advertencia.
          - Verifica si el mensaje indica el fin de una conversación (finishReason == "STOP") o si es un mensaje de usuario. Si es así, llama a finalizeMessageGroup para guardar el mensaje en el historial permanente (chatHistoryLog).

- **Línea 1**: toGroupId(String chatId, Message message):
      - **Propósito**:Crea un identificador único (groupId) para la sesión, combinando chatId y messageId.
      - **Uso**: Asegura que cada sesión tenga un identificador único basado en la conversación y el mensaje.

- **Línea 1**: getProperty(Message message, String key):
      - **Propósito**: Recupera el valor de una propiedad específica de message, como finishReason o id.
      - **Uso**: Facilita el acceso a propiedades en el metadata del mensaje.

- **Línea 1**: finalizeMessageGroup(String chatId, String groupId):

Propósito: Mueve los mensajes agrupados temporalmente desde messageAggregations al historial permanente en chatHistoryLog.
Lógica:
Elimina el grupo de mensajes de messageAggregations.
Si solo hay un mensaje en el grupo, lo guarda directamente en el historial. Si hay varios, combina sus contenidos y luego lo guarda.
Si no hay mensajes en el grupo, registra una advertencia de sesión inactiva.
commitToHistoryLog(String chatId, Message message):

Propósito: Agrega un mensaje al historial permanente chatHistoryLog.
Uso: Llama a este método cuando una sesión se ha finalizado, para registrar el contenido en el historial de conversación.
getAll(String chatId):

Propósito: Devuelve el historial completo de mensajes para un chatId específico.
Uso: Se utiliza en el controlador para recuperar el historial de una conversación completa y enviarlo al modelo de IA para proporcionar contexto en las respuestas.

Ejemplo de Uso en el Segundo Endpoint
En el endpoint generateConversation del ChatController, ChatHistory permite mantener un contexto de conversación. Cada mensaje del usuario se agrega a ChatHistory, y luego se obtiene el historial completo de la conversación antes de enviarlo al modelo de IA. Esto permite que la IA genere una respuesta considerando toda la conversación hasta el momento.