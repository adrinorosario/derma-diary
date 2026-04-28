# LiteRT LM C++ API Documentation Notes

## Headers:

### `runtime/engine/engine.h` (https://github.com/google-ai-edge/LiteRT-LM/blob/main/runtime/engine/engine.h)

This C++ header file defines the starting point for the LiteRT LM C++ API.

What it is trying to achieve:
- Abstracting away the complexities of managing tokens
- Managing the conversation state with the LLM
- Handles the initialization of the model assets, tokenizers, and other specific things like the hardware-specific backends
- Streamling the inference either through one-shot apis or through low-level granular control over the prefill and decoding phases

The main class it  `Engine` which is the highest level of abstraction, where it houses multiple subclasses nested within it, each handling particular functions such as:
- hosting the internal state of the of each separate interaction
- the interaction history of the LLM with each user
- task controllers for handling async execution
- destructors for clearing memory at particular points
- loading and handling model assets for vision and audio tasks

The `Session` subclass handles a single conversation thread of the engine, where it houses:
- the interaction history of the conversation
- pipelines that handle the data and tasks from prompt input to output generation

The `TaskController` subclass performs the following:
- enables async operations/methods to monitor background tasks
- functions that block other processes from running until the current process finsihes execution; also provides method to cancel current inference process and save resources

#### How it would be used here (example workflow):

1. Create the model assets: this would involve pointing the engine to the model path in the directory and loading the model weights and other assets such as the tokenizer, etc.
2. Initialise the engine using the desired hardware backend and other settings such as handling the vision inputs for the images
3. Start the session where the user
4. Generate to get responses from the model

### `runtime/conversation/conversation.h` (https://github.com/google-ai-edge/LiteRT-LM/blob/main/runtime/conversation/conversation.h)

This header handles the function of handling the configurations for the model's conversation instance. Can use default config or build custom configs. 

It performs the following:
- manages the conversation history to remember what the conversation is about
- uses jinja style templates to encode roles and messages as needed for better output generation
- allows for handling context and allowing cloning a conversation or removing unwanted thoughts from the model to free up memory

`ConversationConfig` is the high-level class that handles the process of creating the configuration from the given engine. Contains `Builder` class for creating the `ConversationConfig`. 

Class arguments include:
- the engine instance used to validate `SessionConfig`
- the `SessionConfig` used to create `ConversationConfig`
- option `preface` for the conversation such as background context, tool use, and other extra context
- overwriting the default jinja prompt template if not provided with any
- the channels configured for the conversation
- etc. 

### `runtime/components/model_resources.h` (https://github.com/google-ai-edge/LiteRT-LM/blob/main/runtime/components/model_resources.h)

Handles the loaded model resources the executor needs to prevent the model from being destroyed.
Allows models to be loaded in a lazy way; creates the models when it is actually used and not when it is idle or not in function.
Physical memory is allocated only when the model is used

Performs the following functions:
- allows lifecycle management by loaded model resources
- lazy loading for energy and memory efficiency
- handles distinctions between models that need to run on GPU vs NPU vs CPU, etc.
