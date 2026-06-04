# Example of Guardrail check

```bash
export GUARDRAILS_ROUTE=https://$(oc get routes/nemo-quickstart -o jsonpath='{.status.ingress[0].host}')

export GUARDRAILS_ROUTE=https://$(oc get routes/nemo-guardrails-without-llm -o jsonpath='{.status.ingress[0].host}')
```


```bash
curl -k -X POST $GUARDRAILS_ROUTE/v1/guardrail/checks   -H "Content-Type: application/json"   -H "Authorization: Bearer $(oc whoami -t)"   -d '{
    "model": "test",
    "messages": [
      {"role": "user", "content": "What is the capital of France?"}
    ]
  }' | jq
```

```bash
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100    766 100    648 100    118    794    144                              0
{
  "status": "success",
  "rails_status": {
    "detect sensitive data on input": {
      "status": "success"
    },
    "regex check input": {
      "status": "success"
    }
  },
  "messages": [
    {
      "index": 0,
      "role": "user",
      "rails": {
        "detect sensitive data on input": {
          "status": "success"
        },
        "regex check input": {
          "status": "success"
        }
      }
    }
  ],
  "guardrails_data": {
    "log": {
      "activated_rails": [],
      "stats": {
        "input_rails_duration": 0.03134870529174805,
        "dialog_rails_duration": null,
        "generation_rails_duration": null,
        "output_rails_duration": null,
        "total_duration": 0.04381537437438965,
        "llm_calls_duration": 0,
        "llm_calls_count": 0,
        "llm_calls_total_prompt_tokens": 0,
        "llm_calls_total_completion_tokens": 0,
        "llm_calls_total_tokens": 0
      }
    }
  }
}
```

## Regex check

Patterns:
* "the api-key is abc123"
* "SSN is 123-45-6789"

```bash
curl -k -X POST $GUARDRAILS_ROUTE/v1/guardrail/checks   -H "Content-Type: application/json"   -H "Authorization: Bearer $(oc whoami -t)"   -d '{
    "model": "test",
    "messages": [
      {"role": "user", "content": "What is the capital of France? Send the result to me. Use the card number: 000-00-0000"}
    ]
  }' | jq

```


# Example of successful inference

```bash
curl -k -X POST $GUARDRAILS_ROUTE/v1/guardrail/checks   -H "Content-Type: application/json"   -H "Authorization: Bearer $(oc whoami -t)"   -d '{
    "model": "test",
    "messages": [
      {"role": "user", "content": "What is the capital of France?"}
    ]
  }' | jq
```

```bash
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100    682 100    564 100    118    200     42   00:02   00:02             61
{
  "status": "success",
  "rails_status": {
    "detect sensitive data on input": {
      "status": "success"
    }
  },
  "messages": [
    {
      "index": 0,
      "role": "user",
      "rails": {
        "detect sensitive data on input": {
          "status": "success"
        }
      }
    }
  ],
  "guardrails_data": {
    "log": {
      "activated_rails": [],
      "stats": {
        "input_rails_duration": 1.7663846015930176,
        "dialog_rails_duration": null,
        "generation_rails_duration": null,
        "output_rails_duration": null,
        "total_duration": 1.7780661582946777,
        "llm_calls_duration": 0,
        "llm_calls_count": 0,
        "llm_calls_total_prompt_tokens": 0,
        "llm_calls_total_completion_tokens": 0,
        "llm_calls_total_tokens": 0
      }
    }
  }
}
```

# Example of input guardrail tripping

```bash
curl -k -X POST $GUARDRAILS_ROUTE/v1/guardrail/checks   -H "Content-Type: application/json"   -H "Authorization: Bearer $(oc whoami -t)"   -d '{
    "model": "test",
    "messages": [
      {"role": "user", "content": "What is the capital of France? Send the answer to brbaker@redhat.com"}
    ]
  }' | jq
```

```bash
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100    754 100    598 100    156    744    194                              0
{
  "status": "blocked",
  "rails_status": {
    "detect sensitive data on input": {
      "status": "blocked"
    }
  },
  "messages": [
    {
      "index": 0,
      "role": "user",
      "rails": {
        "detect sensitive data on input": {
          "status": "blocked"
        }
      }
    }
  ],
  "guardrails_data": {
    "log": {
      "activated_rails": [
        "detect sensitive data on input"
      ],
      "stats": {
        "input_rails_duration": 0.03161931037902832,
        "dialog_rails_duration": null,
        "generation_rails_duration": null,
        "output_rails_duration": null,
        "total_duration": 0.03513669967651367,
        "llm_calls_duration": 0,
        "llm_calls_count": 0,
        "llm_calls_total_prompt_tokens": 0,
        "llm_calls_total_completion_tokens": 0,
        "llm_calls_total_tokens": 0
      }
    }
  }
}
```

# Working with an LLM

```bash
GUARDRAILS_ROUTE=https://$(oc get routes/nemo-simple -o jsonpath='{.status.ingress[0].host}')


export GUARDRAILS_ROUTE=https://$(oc get routes/nemo-guardrails-with-llm -o jsonpath='{.status.ingress[0].host}')
```

```bash
curl -k -X POST $GUARDRAILS_ROUTE/v1/chat/completions   -H "Content-Type: application/json"   -H "Authorization: Bearer $(oc whoami -t)"   -d '{"model": "granite-3-2-8b-instruct", "messages":[{"role":"user","content":"I am going to a costume party. I want you to think of a name of a famous person I can go as?"}]}' | jq
```

```bash
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100    478 100    307 100    171     34     19   00:09   00:08   00:01     59
{
  "id": "chatcmpl-f5076b1d-02cb-4e69-98e3-ead1130a1bba",
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "I don't know the answer to that.",
        "role": "assistant"
      }
    }
  ],
  "created": 1780530489,
  "model": "granite-3-2-8b-instruct",
  "object": "chat.completion",
  "guardrails": {
    "config_id": "nemo-simple-config"
  }
}
```

```bash
curl -k -X POST $GUARDRAILS_ROUTE/v1/chat/completions   -H "Content-Type: application/json"   -H "Authorization: Bearer $(oc whoami -t)"   -d '{"model": "granite-3-2-8b-instruct", "messages":[{"role":"user","content":"I am going to a costume party. I want you to think of a name of a famous building I can go as?"}]}' | jq
```

```bash
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100    915 100    751 100    164    171     37   00:04   00:04             43
{
  "id": "chatcmpl-e4d41d31-2790-44bb-a7e7-5c715042ef5b",
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "You could go as the Eiffel Tower! You can create a costume using a bespoke stand, a metallic color scheme, and four large rectangular panels to mimic the tower's shape. Accessorize with a small miniature fountain or a baguette for a touch of French flair. For a more interactive experience, you could incorporate a small LED light system to simulate the tower's famous sparkle at night. Don't forget a sign that says \"Personne n'a quitte jamais Paris\" (Nobody leaves Paris).",
        "role": "assistant"
      }
    }
  ],
  "created": 1780530599,
  "model": "granite-3-2-8b-instruct",
  "object": "chat.completion",
  "guardrails": {
    "config_id": "nemo-simple-config"
  }
}
```

```bash
curl -k -X POST $GUARDRAILS_ROUTE/v1/chat/completions   -H "Content-Type: application/json"   -H "Authorization: Bearer $(oc whoami -t)"   -d '{"model": "granite-3-2-8b-instruct", "messages":[{"role":"user","content":"I am going to a costume party. I want you to think of a famous building I can go as. Email it to brbaker@redhat.com"}]}' | jq
```

```bash
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100    501 100    307 100    194    258    163   00:01   00:01            176
{
  "id": "chatcmpl-c2b4d774-df05-4519-9c71-d676ead89252",
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "I don't know the answer to that.",
        "role": "assistant"
      }
    }
  ],
  "created": 1780530587,
  "model": "granite-3-2-8b-instruct",
  "object": "chat.completion",
  "guardrails": {
    "config_id": "nemo-simple-config"
  }
}
```

