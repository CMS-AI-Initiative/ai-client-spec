# Requirements

This document outlines the requirements for the language agnostic AI client specification. The concrete technical architecture will be defined and outlined in a separate document.

## Objective

Enable calling any generative AI implementation using a uniform API in various programming languages.

### Context

* This project defines a specification for an AI client SDK that allows calling models of any provider.
* The initial background for starting this project is the lack of such an SDK in the PHP-based CMS ecosystem. Since UI on the web today heavily relies on JavaScript (in addition to whichever server-side language), the SDK's API needs to be centered in PHP, but accessible via JavaScript as well (e.g. through a REST API).
* While a few noteworthy projects with a related purpose exist in various programming languages, they are not in PHP, or their API is not provider agnostic, or their API lacks flexibility for emerging modalities and features.
* Given the specification is already meant to apply in both PHP and client-side JavaScript, it needs to follow paradigms that can be expressed in both programming languages. As such, a stretch goal in the longer-term future could be to implement the same specification for other programming languages as well, e.g. Go, Python, or Ruby.

## Architecture requirements

This section lists the key requirements that the specification and any implementation of it must meet. For explanation on specific terms, see the [glossary](./GLOSSARY.md).

* MUST support any kinds of AI implementation, i.e. cloud-based AI, server-side AI, client-side AI.
* MUST define clear, language-agnostic data structures for AI inputs and outputs, precise enough to allow for consistent implementation across languages (e.g. express in JSON schema).
* MUST support arbitrary combinations of input and output modalities, regardless of which combinations or singular modalities generative AI models support today. Such as (non comprehensive list):
  * Text
  * Function call
  * Image
  * Audio
  * Video
* MUST support additional non-generative features such as (non comprehensive list):
  * Classification
  * Text to Speech
  * Embedding
* MUST support response streaming for arbitrary output modalities.
* MUST allow for long-running operations that may take several minutes, _if_ relevant for the selected provider.
* MUST define standard ways for interacting with optional provider capabilities, such as managing chat history or specifying multimodal inputs/outputs, _if_ the selected provider supports them.
* MUST support diverse common model parameters, such as temperature, top P, or image aspect ratio, with uniform names and behavior across providers, _if_ the selected provider supports them.
* MUST define a modular component model that allows for the addition of new providers, models, and features without modifying core functionality.
* MUST define an API for external packages to register and implement AI model providers.
* MUST be decoupled from any AI provider's implementation details (e.g. not all providers require HTTP requests or API authentication).
* MUST allow provider and model discovery based on specific inputs, outputs, and configuration options supported.
* MUST define data types and interfaces that have direct equivalents in all supported languages (e.g. no multiple inheritance for classes).
* MUST define separate APIs for SDK usage and provider registration, so that iterations or breaking changes in one don't automatically affect the other.
* MUST include a REST API specification to allow accessing the AI provider and model capabilities through client-side code (e.g. JavaScript).

### Best practices

* SHOULD define concepts and paradigms so that they can be applied in other AI infrastructure projects, either in combination or separate from the AI client SDK (e.g. MCP, real-time AI abstraction, prompt generation).
* SHOULD provide middleware that can be used to "polyfill" certain functionality when a provider does not support it (e.g. message history, downloading files from URLs).
* SHOULD allow for arbitrary request and response parameters for specific providers or models to be passed through even when not formally supported, to cater for provider specific features or to allow for newly added features to be used before official support is added to the SDK.

### Out of scope

* MUST NOT include any common AI features beyond the actual AI client (e.g. no MCP, no agents).
* MUST NOT include any actual provider implementations in the core package—these should be separate packages.
* MUST NOT include a real-time / live AI abstraction as it requires different infrastructure.

## Credit

This project is heavily based on researching existing AI providers and existing AI client SDKs, and it takes significant learnings from these into account for the specification. All of these products helped inform the language agnostic requirements.

Below is a list of products that were reviewed. The list is non comprehensive, based on a best-effort approach to include all key resources reviewed.

### Cloud-based AI providers

_(in alphabetical order)_

* [Anthropic API](https://docs.anthropic.com/en/api/)
* [fal API](https://docs.fal.ai/model-endpoints)
* [Google Generative Language API](https://ai.google.dev/api/all-methods)
* [Google Vertex AI API](https://cloud.google.com/vertex-ai/docs/reference/rest)
* [Nvidia LLM API](https://docs.api.nvidia.com/nim/reference/llm-apis)
* [OpenAI API](https://platform.openai.com/docs/api-reference/)
* [Perplexity API](https://docs.perplexity.ai/api-reference/)
* [Replicate API](https://replicate.com/docs/reference/http)
* [X AI API](https://docs.x.ai/docs/api-reference)

### AI client SDKs

_(in alphabetical order)_

* [AI Services WordPress plugin](https://github.com/felixarntz/ai-services)
* [Drupal AI](https://git.drupalcode.org/project/ai)
* [Firebase Genkit](https://github.com/firebase/genkit)
* [Google Gen AI SDK for TypeScript and JavaScript](https://github.com/googleapis/js-genai)
* [LangChain.js](https://github.com/langchain-ai/langchainjs)
* [LLPhant](https://github.com/LLPhant/LLPhant)
* [OpenAI PHP](https://github.com/openai-php/client)
* [Vercel AI SDK](https://github.com/vercel/ai)

### AI specifications

* [A2A protocol](https://github.com/google/A2A)
* [MCP](https://github.com/modelcontextprotocol/modelcontextprotocol)
* [OpenAI model spec](https://cdn.openai.com/spec/model-spec-2024-05-08.html)

### Client-side AI

_(in alphabetical order)_

* [Chrome built-in AI Prompt API](https://github.com/webmachinelearning/prompt-api)
* [transformers.js](https://github.com/huggingface/transformers.js/)
