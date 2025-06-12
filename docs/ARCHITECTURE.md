# Architecture

This document outlines the architecture for the language agnostic AI client specification. It is critical that it meets all [requirements](./REQUIREMENTS.md).

**Note:** As the primary focus for this specification is to facilitate AI usage in PHP, some of the underlying syntax used in this documentation may use PHP conventions. This does not hinder interoperability with other programming languages. Only PHP language capabilities that can be easily replicated in other relevant programming languages will be used.

## High-level API design

The API design at a high level is heavily inspired by the [Vercel AI SDK](https://github.com/vercel/ai), which is widely used in the NodeJS ecosystem and one of the very few comprehensive AI client SDKs available.

The main additional aspect that the Vercel AI SDK does not cater for easily is for a developer to use AI in a way that the choice of provider remains with the user. To clarify with an example: Instead of "Generate text with Google's model `gemini-2.5-flash`", go with "Generate text using any provider model that supports text generation and multimodal input". In other words, there needs to be a mechanism that allows finding any configured model that supports the given set of required AI features and capabilities.

### Code examples

#### Generate text using a Google model

```php
$text = Ai::generateText(
    'Write a 2-verse poem about PHP.',
    Google::model('gemini-2.5-flash')
);
```

#### Generate multiple text candidates using an Anthropic model

```php
$result = Ai::generateTextResult(
    'Write a 2-verse poem about PHP.',
    Anthropic::model(
        'claude-3.7-sonnet',
        [TextGenerationConfig::CANDIDATE_COUNT => 4]
    )
);
$texts = CandidatesUtil::toTexts(
    $result->getCandidates()
);
```

#### Generate an image using any suitable OpenAI model

```php
$modelsMetadata = Ai::defaultRegistry()->findProviderModelsMetadataForSupport(
    'openai',
    AiFeature::IMAGE_GENERATION
);
$imageFile = Ai::generateImage(
    'Generate an illustration of the PHP elephant in the Carribean sea.',
    Ai::defaultRegistry()->getProviderModel(
        'openai',
        $modelsMetadata[0]->getId()
    )
);
```

#### Generate an image using any suitable model from any provider

```php
$providerModelsMetadata = Ai::defaultRegistry()->findModelsMetadataForSupport(
    AiFeature::IMAGE_GENERATION
);
$imageFile = Ai::generateImage(
    'Generate an illustration of the PHP elephant in the Carribean sea.',
    Ai::defaultRegistry()->getProviderModel(
        $providerModelsMetadata[0]->getProvider()->getId(),
        $providerModelsMetadata[0]->getModels()[0]->getId()
    )
);
```

#### Generate text with JSON output using any suitable model from any provider

```php
$providerModelsMetadata = Ai::defaultRegistry()->findModelsMetadataForSupport(
    AiFeature::TEXT_GENERATION,
    [
        // Make sure the model supports JSON output as well as following a given schema.
        TextGenerationConfig::OUTPUT_MIME_TYPE => 'application/json',
        TextGenerationConfig::OUTPUT_SCHEMA    => true,
    ]
);
$imageFile = Ai::generateText(
    'Transform the following CSV content into a JSON array of row data.',
    Ai::defaultRegistry()->getProviderModel(
        $providerModelsMetadata[0]->getProvider()->getId(),
        $providerModelsMetadata[0]->getModels()[0]->getId(),
        [
            AiModelConfig::GENERATION_CONFIG => [
                TextGenerationConfig::OUTPUT_MIME_TYPE => 'application/json',
                TextGenerationConfig::OUTPUT_SCHEMA    => [
                    'type'  => 'array',
                    'items' => [
                        'type'       => 'object',
                        'properties' => [
                            'name' => [
                                'type' => 'string',
                            ],
                            'age'  => [
                                'type' => 'integer',
                            ],
                        ],
                    ],
                ],
            ],
        ]
    )
);
```

## Class diagrams

This section shows comprehensive class diagrams for the proposed architecture. For explanation on specific terms, see the [glossary](./GLOSSARY.md).

**Note:** The class diagrams are not meant to be entirely comprehensive in terms of which AI features and capabilities are or will be supported. For now, they simply use "text generation", "image generation", "text to speech", and "speech generation" for illustrative purposes. Other features like "music generation" or "video generation" etc. would work similarly.

**Note:** The class diagrams are also not meant to be comprehensive in terms of any specific configuration keys or parameters which are or will be supported. For now, the relevant definitions don't include any specific parameter names or constants.

### Full class diagram

```mermaid
---
config:
  class:
    hideEmptyMembersBox: true
---
classDiagram
direction LR
    namespace Ai {
        class AiEntrypoint {
            +defaultRegistry() AiProviderRegistry
            +isConfigured(AiProviderAvailability $availability) bool$
            +generateText(string|MessagePart|MessagePart[]|Message|Message[] $prompt, AiModel $model) string$
            +streamGenerateText(string|MessagePart|MessagePart[]|Message|Message[] $prompt, AiModel $model) Generator< string >$
            +generateImage(string|MessagePart|MessagePart[]|Message|Message[] $prompt, AiModel $model) File$
            +textToSpeech(string|MessagePart|MessagePart[]|Message|Message[] $prompt, AiModel $model) File$
            +generateSpeech(string|MessagePart|MessagePart[]|Message|Message[] $prompt, AiModel $model) File$
            +generateResult(string|MessagePart|MessagePart[]|Message|Message[] $prompt, AiModel $model) GenerativeAiResult$
            +generateOperation(string|MessagePart|MessagePart[]|Message|Message[] $prompt, AiModel $model) GenerativeAiOperation$
            +generateTextResult(string|MessagePart|MessagePart[]|Message|Message[] $prompt, AiModel $model) GenerativeAiResult$
            +streamGenerateTextResult(string|MessagePart|MessagePart[]|Message|Message[] $prompt, AiModel $model) Generator< GenerativeAiResult >$
            +generateImageResult(string|MessagePart|MessagePart[]|Message|Message[] $prompt, AiModel $model) GenerativeAiResult$
            +textToSpeechResult(string|MessagePart|MessagePart[]|Message|Message[] $prompt, AiModel $model) GenerativeAiResult$
            +generateSpeechResult(string|MessagePart|MessagePart[]|Message|Message[] $prompt, AiModel $model) GenerativeAiResult$
            +generateTextOperation(string|MessagePart|MessagePart[]|Message|Message[] $prompt, AiModel $model) GenerativeAiOperation$
            +generateImageOperation(string|MessagePart|MessagePart[]|Message|Message[] $prompt, AiModel $model) GenerativeAiOperation$
            +textToSpeechOperation(string|MessagePart|MessagePart[]|Message|Message[] $prompt, AiModel $model) GenerativeAiOperation$
            +generateSpeechOperation(string|MessagePart|MessagePart[]|Message|Message[] $prompt, AiModel $model) GenerativeAiOperation$
        }
    }
    namespace Ai.Types {
        class Message {
            +getRole() MessageRole
            +getParts() MessagePart[]
            +getJsonSchema() array< string, mixed >$
        }
        class MessagePart {
            +getType() MessagePartType
            +getText() string?
            +getInlineFile() InlineFile?
            +getRemoteFile() RemoteFile?
            +getFunctionCall() FunctionCall?
            +getFunctionResponse() FunctionResponse?
            +getJsonSchema() array< string, mixed >$
        }
        class File {
        }
        class InlineFile {
            +getMimeType() string
            +getBase64Data() string
            +getJsonSchema() array< string, mixed >$
        }
        class RemoteFile {
            +getMimeType() string
            +getUrl() string
            +getJsonSchema() array< string, mixed >$
        }
        class LocalFile {
            +getMimeType() string
            +getPath() string
            +getJsonSchema() array< string, mixed >$
        }
        class FunctionCall {
            +getId() string
            +getName() string
            +getArgs() array< string, mixed >
            +getJsonSchema() array< string, mixed >$
        }
        class FunctionResponse {
            +getId() string
            +getName() string
            +getResponse() mixed
            +getJsonSchema() array< string, mixed >$
        }
        class Operation {
            +getId() string
            +getState() OperationState
            +getJsonSchema() array< string, mixed >$
        }
        class GenerativeAiOperation {
            +getId() string
            +getState() OperationState
            +getResult() GenerativeAiResult
            +getJsonSchema() array< string, mixed >$
        }
        class GenerativeAiResult {
            +getId() string
            +getCandidates() Candidate[]
            +getUsage() TokenUsage
            +getJsonSchema() array< string, mixed >$
        }
        class Candidate {
            +getMessage() Message
            +getFinishReason() FinishReason
            +getTokenCount() int
            +getJsonSchema() array< string, mixed >$
        }
        class TokenUsage {
            +getPromptTokens() int
            +getCompletionTokens() int
            +getTotalTokens() int
            +getJsonSchema() array< string, mixed >$
        }
    }
    namespace Ai.Types.Enums {
        class MessageRole {
            USER
            MODEL
            SYSTEM
        }
        class MessagePartType {
            TEXT
            INLINE_FILE
            REMOTE_FILE
            FUNCTION_CALL
            FUNCTION_RESPONSE
        }
        class FinishReason {
            STOP
            LENGTH
            CONTENT_FILTER
            TOOL_CALLS
            ERROR
        }
        class OperationState {
            STARTING
            PROCESSING
            SUCCEEDED
            FAILED
            CANCELED
        }
        class AiModality {
            TEXT
            DOCUMENT
            IMAGE
            AUDIO
            VIDEO
        }
    }
    namespace Ai.Util {
        class MessageUtil {
            +toText(Message $message) string$
            +toImageFile(Message $message) File$
            +toAudioFile(Message $message) File$
            +toVideoFile(Message $message) File$
        }
        class CandidatesUtil {
            +toTexts(Candidate[] $candidates) string[]$
            +toImageFiles(Candidate[] $candidates) File[]$
            +toAudioFiles(Candidate[] $candidates) File[]$
            +toVideoFiles(Candidate[] $candidates) File[]$
            +toFirstText(Candidate[] $candidates) string$
            +toFirstImageFile(Candidate[] $candidates) File$
            +toFirstAudioFile(Candidate[] $candidates) File$
            +toFirstVideoFile(Candidate[] $candidates) File$
        }
    }
    namespace Ai.Providers {
        class AiProviderRegistry {
            +registerProvider(string $className) void
            +hasProvider(string $idOrClassName) bool
            +getProviderClassName(string $id) string
            +isProviderConfigured(string $idOrClassName) bool
            +getProviderModel(string $idOrClassName, string $modelId, AiModelConfig|array $modelConfig) AiModel
            +findProviderModelsMetadataForSupport(string $idOrClassName, AiFeature $feature, array<string, mixed > $capabilities) AiModelMetadata[]
            +findModelsMetadataForSupport(AiFeature $feature, array<string, mixed > $capabilities) AiProviderModelMetadata[]
        }
    }
    namespace Ai.Providers.Contracts {
        class AiProvider {
            +metadata() AiProviderMetadata$
            +model(string $modelId, AiModelConfig|array< string, mixed > $modelConfig) AiModel$
            +availability() AiProviderAvailability$
            +modelMetadataDirectory() AiModelMetadataDirectory$
        }
        class AiModel {
            +metadata() AiModelMetadata
            +setConfig(AiModelConfig $config) void
            +getConfig() AiModelConfig
            +getSupportedCapabilities() AiCapability[]$
        }
        class AiProviderAvailability {
            +isConfigured() bool
        }
        class AiModelMetadataDirectory {
            +listModelMetadata() AiModelMetadata[]
            +hasModelMetadata(string $modelId) bool
            +getModelMetadata(string $modelId) AiModelMetadata
        }
        class WithGenerativeAiOperations {
            +getOperation(string $operationId) GenerativeAiOperation
        }
        class AiTextGenerationModel {
            +generateTextResult(Message[] $prompt) GenerativeAiResult
            +streamGenerateTextResult(Message[] $prompt) Generator< GenerativeAiResult >
        }
        class AiImageGenerationModel {
            +generateImageResult(Message[] $prompt) GenerativeAiResult
        }
        class AiTextToSpeechModel {
            +textToSpeechResult(Message[] $prompt) GenerativeAiResult
        }
        class AiSpeechGenerationModel {
            +generateSpeechResult(Message[] $prompt) GenerativeAiResult
        }
        class AiTextGenerationOperationModel {
            +generateTextOperation(Message[] $prompt) GenerativeAiOperation
        }
        class AiImageGenerationOperationModel {
            +generateImageOperation(Message[] $prompt) GenerativeAiOperation
        }
        class AiTextToSpeechOperationModel {
            +textToSpeechOperation(Message[] $prompt) GenerativeAiOperation
        }
        class AiSpeechGenerationOperationModel {
            +generateSpeechOperation(Message[] $prompt) GenerativeAiOperation
        }
        class WithHttpClient {
            +setHttpClient(HttpClient $client) void
            +getHttpClient() HttpClient
        }
        class HttpClient {
            +send(RequestInterface $request, array< string, mixed > $options) ResponseInterface
            +request(string $method, string $uri, array< string, mixed > $options) ResponseInterface
        }
        class WithAuthentication {
            +setAuthentication(Authentication $authentication) void
            +getAuthentication() Authentication
        }
        class Authentication {
            +authenticate(RequestInterface $request) void
            +getJsonSchema() array< string, mixed >$
        }
    }
    namespace Ai.Providers.Types {
        class AiProviderMetadata {
            +getId() string
            +getName() string
            +getType() AiProviderType
            +getJsonSchema() array< string, mixed >$
        }
        class AiModelMetadata {
            +getId() string
            +getName() string
            +getSupportedFeatures() AiFeature[]
            +getSupportedCapabilities() AiCapability[]
            +getJsonSchema() array< string, mixed >$
        }
        class AiProviderModelsMetadata {
            +getProvider() AiProviderMetadata
            +getModels() AiModelMetadata[]
            +getJsonSchema() array< string, mixed >$
        }
        class AiModelConfig {
            +setSystemInstruction(string|MessagePart|MessagePart[]|Message $systemInstruction) void
            +getSystemInstruction() Message?
            +setGenerationConfig(GenerationConfig $config) void
            +getGenerationConfig() GenerationConfig?
            +setTools(Tool[] $tools) void
            +getTools() Tool[]
            +getJsonSchema() array< string, mixed >$
        }
        class GenerationConfig {
            +setValue(string $key, mixed $value) void
            +getValue(string $key) mixed
            +getValues() array< string, mixed >
            +getAdditionalValues() array< string, mixed >
            +getJsonSchema() array< string, mixed >$
        }
        class TextGenerationConfig {
        }
        class ImageGenerationConfig {
        }
        class TextToSpeechConfig {
        }
        class SpeechGenerationConfig {
        }
        class Tool {
            +getType() ToolType
            +getFunctionDeclarations() FunctionDeclaration[]?
            +getWebSearch() WebSearch?
            +getJsonSchema() array< string, mixed >$
        }
        class FunctionDeclaration {
            +getName() string
            +getDescription() string
            +getParameters() mixed
            +getJsonSchema() array< string, mixed >$
        }
        class WebSearch {
            +getAllowedDomains() string[]
            +getDisallowedDomains() string[]
            +getJsonSchema() array< string, mixed >$
        }
        class AiCapability {
            +isSupported() bool
            +isSupportedValue(mixed $value) bool
            +getSupportedValues() mixed[]
            +getJsonSchema() array< string, mixed >$
        }
    }
    namespace Ai.Providers.Enums {
        class AiProviderType {
            CLOUD
            SERVER
            CLIENT
        }
        class ToolType {
            FUNCTION_DECLARATIONS
            WEB_SEARCH
        }
        class AiFeature {
            TEXT_GENERATION
            IMAGE_GENERATION
            TEXT_TO_SPEECH
            SPEECH_GENERATION
            MUSIC_GENERATION
            VIDEO_GENERATION
            EMBEDDING
        }
    }
    namespace Ai.Providers.Util {
        class AiFeaturesUtil {
            +getSupportedFeatures(AiModel|string $modelClass) AiFeature[]$
            +getSupportedCapabilities(AiModel|string $modelClass) AiCapability[]$
        }
    }

    %% Annotations for Ai namespaces, except Ai.Providers.*.
    <<interface>> File
    <<interface>> Operation
    <<Enumeration>> MessageRole
    <<Enumeration>> MessagePartType
    <<Enumeration>> FinishReason
    <<Enumeration>> OperationState
    <<Enumeration>> AiModality

    AiEntrypoint .. AiProviderRegistry : creates
    AiEntrypoint .. Message : receives
    AiEntrypoint .. MessagePart : receives
    AiEntrypoint .. GenerativeAiResult : creates
    AiEntrypoint .. GenerativeAiOperation : creates
    Message "1" *-- "1..*" MessagePart
    MessagePart "1" o-- "0..1" InlineFile
    MessagePart "1" o-- "0..1" RemoteFile
    MessagePart "1" o-- "0..1" FunctionCall
    MessagePart "1" o-- "0..1" FunctionResponse
    GenerativeAiOperation "1" o-- "0..1" GenerativeAiResult
    GenerativeAiResult "1" o-- "1..*" Candidate
    GenerativeAiResult "1" o-- "1" TokenUsage
    Candidate "1" o-- "1" Message
    Message ..> MessageRole
    MessagePart ..> MessagePartType
    Operation ..> OperationState
    GenerativeAiOperation ..> OperationState
    Candidate ..> FinishReason
    File <|-- InlineFile
    File <|-- RemoteFile
    File <|-- LocalFile
    Operation <|-- GenerativeAiOperation

    %% Annotations for Ai.Providers.* namespaces.
    <<interface>> AiProvider
    <<interface>> AiModel
    <<interface>> AiProviderAvailability
    <<interface>> AiModelMetadataDirectory
    <<interface>> WithGenerativeAiOperations
    <<interface>> AiTextGenerationModel
    <<interface>> AiImageGenerationModel
    <<interface>> AiTextToSpeechModel
    <<interface>> AiSpeechGenerationModel
    <<interface>> AiTextGenerationOperationModel
    <<interface>> AiImageGenerationOperationModel
    <<interface>> AiTextToSpeechOperationModel
    <<interface>> AiSpeechGenerationOperationModel
    <<interface>> WithHttpClient
    <<interface>> HttpClient
    <<interface>> WithAuthentication
    <<interface>> Authentication
    <<interface>> GenerationConfig
    <<Enumeration>> AiFeature
    <<Enumeration>> AiProviderType

    AiProvider .. AiModel : creates
    AiProvider "1" *-- "1" AiProviderMetadata
    AiProvider "1" *-- "1" AiProviderAvailability
    AiProvider "1" *-- "1" AiModelMetadataDirectory
    AiModel "1" *-- "1" AiModelMetadata
    AiModel "1" *-- "1" AiModelConfig
    AiProviderModelsMetadata "1" o-- "1" AiProviderMetadata
    AiProviderModelsMetadata "1" o-- "1..*" AiModelMetadata
    AiProviderRegistry "1" o-- "0..*" AiProvider
    AiProviderRegistry "1" o-- "0..*" AiProviderMetadata
    AiModelMetadataDirectory "1" o-- "1..*" AiModelMetadata
    AiModelMetadata "1" o-- "1..*" AiFeature
    AiModelMetadata "1" o-- "0..*" AiCapability
    AiModelConfig "1" o-- "0..1" GenerationConfig
    AiModelConfig "1" o-- "0..*" Tool
    Tool "1" o-- "0..*" FunctionDeclaration
    Tool "1" o-- "0..1" WebSearch
    AiProviderMetadata ..> AiProviderType
    AiModelMetadata ..> AiFeature
    AiModel <|-- AiTextGenerationModel
    AiModel <|-- AiImageGenerationModel
    AiModel <|-- AiTextToSpeechModel
    AiModel <|-- AiSpeechGenerationModel
    AiModel <|-- AiTextGenerationOperationModel
    AiModel <|-- AiImageGenerationOperationModel
    AiModel <|-- AiTextToSpeechOperationModel
    AiModel <|-- AiSpeechGenerationOperationModel
    GenerationConfig <|-- TextGenerationConfig
    GenerationConfig <|-- ImageGenerationConfig
    GenerationConfig <|-- TextToSpeechConfig
    GenerationConfig <|-- SpeechGenerationConfig
    WithGenerativeAiOperations <|-- AiTextGenerationOperationModel
    WithGenerativeAiOperations <|-- AiImageGenerationOperationModel
    WithGenerativeAiOperations <|-- AiTextToSpeechOperationModel
    WithGenerativeAiOperations <|-- AiSpeechGenerationOperation
```

### Class diagram zoomed in on AI usage ONLY

TODO.

### Class diagram zoomed in on AI provider registration and implementation ONLY

TODO.
