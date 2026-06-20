export default {
  async fetch(request, env, ctx) {
    const url = new URL(request.url);

    // CORS preflight
    if (request.method === "OPTIONS") {
      return new Response(null, {
        headers: {
          "Access-Control-Allow-Origin": "*",
          "Access-Control-Allow-Methods": "POST, OPTIONS",
          "Access-Control-Allow-Headers": "Content-Type, Authorization",
        },
      });
    }

    // Main endpoint: /v1/chat/completions
    if (request.method === "POST" && url.pathname === "/v1/chat/completions") {
      try {
        const body = await request.json();
        const { messages, model, temperature = 0.7, max_tokens = 1024 } = body;

        if (!messages || !model) {
          return new Response(
            JSON.stringify({ error: "Missing messages or model" }),
            { status: 400 }
          );
        }

        // Convert messages to prompt string
        let prompt = "";
        for (const msg of messages) {
          prompt += `${msg.role}: ${msg.content}\n`;
        }

        // Call Cloudflare Workers AI
        const response = await env.AI.run(
          "@cf/meta/llama-3-8b-instruct",
          {
            prompt: prompt,
            max_tokens: max_tokens,
            temperature: temperature,
          }
        );

        // Return OpenAI format
        return new Response(
          JSON.stringify({
            id: "chatcmpl_" + Math.random().toString(36).substring(7),
            object: "chat.completion",
            created: Math.floor(Date.now() / 1000),
            model: model,
            choices: [
              {
                index: 0,
                message: {
                  role: "assistant",
                  content: response.result.response,
                },
                finish_reason: "stop",
              },
            ],
            usage: {
              prompt_tokens: 100,
              completion_tokens: 50,
              total_tokens: 150,
            },
          }),
          {
            headers: {
              "Content-Type": "application/json",
              "Access-Control-Allow-Origin": "*",
            },
          }
        );
      } catch (error) {
        return new Response(
          JSON.stringify({ error: error.message }),
          { status: 500 }
        );
      }
    }

    // Health check
    if (url.pathname === "/health") {
      return new Response(JSON.stringify({ status: "ok" }));
    }

    return new Response("Not Found", { status: 404 });
  },
};
