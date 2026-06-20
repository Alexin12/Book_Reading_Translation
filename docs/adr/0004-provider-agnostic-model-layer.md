# Provider-agnostic model abstraction layer

All LLM calls go through one abstraction layer (e.g. LiteLLM) so the same code can target OpenAI, DeepSeek, Gemini, or Claude. The user supplies an API key and selects a model; new providers cost almost nothing to add. Model selection and API keys are stored per user, so a future multi-user version where each user brings their own key needs no rework. This same layer is reused for the future Explanation feature.
