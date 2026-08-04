---
name: Run a streaming DataGol AI conversation
description: Open a conversation on the Events.com DataGol AI service, send a message to the streaming agent, consume the SSE run, and stop or read it back.
api: openapi/eventscom-datagol-ai-openapi.yml
generated: '2026-08-04'
method: generated
operations:
  - create_conversation_ai_api_v2_conversations_post
  - send_message_to_streaming_agent_ai_api_v2_messages_streaming_post
  - stream_message_events_ai_api_v2_messages_streaming__conversation_id___message_id__get
  - stop_streaming_agent_run_ai_api_v2_messages__conversation_id___message_id__stop_post
  - list_active_runs_ai_api_v2_runs_active_get
  - get_messages_by_conversation_id_ai_api_v2_conversations__conversation_id__messages_get
  - post_message_feedback_ai_api_v2_feedback_post
---

# Run a streaming DataGol AI conversation

The DataGol AI service hosts Events.com's analytics agent. A run is asynchronous: you post a message, then
consume a server-sent-event stream keyed by `conversation_id` + `message_id`.

## Before you start

- **Base URL.** `https://datagol-ai.events.com/`
- **Auth.** This service declares **no** `securityScheme` in its OpenAPI at all, and its spec is served
  anonymously at `/openapi.json`. Do not read that as "no auth required" — treat the JWT you use against the
  platform API as required, and confirm with Events.com before building on it.
- **Errors.** This is FastAPI: validation failures return `422` with
  `{"detail":[{"loc":[...],"msg":"...","type":"..."}]}`. No other 4xx is declared.
- **No idempotency key.** A retried `POST` starts a *second* agent run. Always check
  `list_active_runs_ai_api_v2_runs_active_get` before re-sending.

## Steps

1. **Create a conversation** — `create_conversation_ai_api_v2_conversations_post`
   `POST /ai/api/v2/conversations` → keep the `conversation_id`.

2. **Send a message to the streaming agent** —
   `send_message_to_streaming_agent_ai_api_v2_messages_streaming_post`
   `POST /ai/api/v2/messages/streaming` → returns the `message_id` for the run.

   For a non-streaming, single-shot completion use
   `post_completion_generic_ai_api_v2_messages_complete_post` (`POST /ai/api/v2/messages/complete`)
   instead.

3. **Consume the stream** —
   `stream_message_events_ai_api_v2_messages_streaming__conversation_id___message_id__get`
   `GET /ai/api/v2/messages/streaming/{conversation_id}/{message_id}`
   Send `Accept: text/event-stream`. Read to completion; do not poll this endpoint.

4. **Cancel a run that is taking too long** —
   `stop_streaming_agent_run_ai_api_v2_messages__conversation_id___message_id__stop_post`
   `POST /ai/api/v2/messages/{conversation_id}/{message_id}/stop`

5. **Read the transcript back** —
   `get_messages_by_conversation_id_ai_api_v2_conversations__conversation_id__messages_get`
   `GET /ai/api/v2/conversations/{conversation_id}/messages`

6. **Record quality feedback** — `post_message_feedback_ai_api_v2_feedback_post`
   `POST /ai/api/v2/feedback`

## Avoid the deprecated path

The entire `Home Chat Agent APIs` group — `/ai/api/v2/home-chat-agent/*`, including
`run_ai_api_v2_home_chat_agent_run_post` and its conversation CRUD — is marked `deprecated: true`, as is
`get_conversation_by_id_ai_api_v2_conversations__conversation_id__get`. Events.com publishes no sunset date
and names no successor, so prefer the `/ai/api/v2/conversations` + `/ai/api/v2/messages` pair above and pin
your integration tests.

## Related surfaces

- Python analysis agent: `openapi/eventscom-datagol-python-agent-openapi.yml`
  (`query_agent_unified_ai_api_v2_python_agent_query_post`).
- MCP tool listing: `list_mcp_tools_ai_api_v2_mcp_tools_post` (`POST /ai/api/v2/mcp/tools`).
