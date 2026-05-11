# Polling Feature - Agora.io

## Overview

Agora oferece um **Polling Feature integrado** na Customization API. Permite criar votações em tempo real dentro de uma chamada de vídeo, com sincronização automática entre participantes.

**Casos de uso:**
- Votações durante assembleia
- Pesquisas de satisfação
- Eleições (com regras customizadas)
- Quiz interativo

---

## Como Funciona

### Fluxo Básico

1. **Host cria poll** → Define pergunta + opções
2. **Envia aos attendees** → Via custom event (POLLS)
3. **Attendees votam** → Selecionam opção
4. **Host vê resultados** → Em tempo real, percentuais
5. **Host fecha poll** → Mostra resultado final

### Estrutura de Dados

```json
{
  "id": "poll_001",
  "status": "ACTIVE",
  "question": "Quem vota em A?",
  "answers": null,
  "options": [
    {
      "text": "Opção A",
      "value": "a",
      "votes": [
        { "uid": 12345, "timestamp": 1234567890 },
        { "uid": 67890, "timestamp": 1234567891 }
      ],
      "percent": "66"
    },
    {
      "text": "Opção B",
      "value": "b",
      "votes": [
        { "uid": 11111, "timestamp": 1234567892 }
      ],
      "percent": "33"
    }
  ],
  "multiple_response": false,
  "createdBy": 12345
}
```

---

## Setup: Adicionar Poll Button

### Step 1: Criar Bottom Toolbar com Poll Button

```jsx
// customization/CustomBottomToolBar.tsx
import {
  ToolbarPreset,
  ToolbarItem,
  TertiaryButton,
  useSidePanel,
} from "customization-api";
import React from "react";

export const POLL_SIDEBAR_NAME = "poll-side-pane";

const PollButtonWithSidePanel = () => {
  const { setSidePanel } = useSidePanel();

  return (
    <ToolbarItem style={{ position: "relative" }}>
      <TertiaryButton
        containerStyle={{ padding: 10 }}
        textStyle={{ fontWeight: "600", fontSize: 14, color: "white" }}
        text="Poll"
        onPress={() => {
          setSidePanel(POLL_SIDEBAR_NAME);
        }}
      />
    </ToolbarItem>
  );
};

const CustomBottomBar = () => {
  return (
    <ToolbarPreset
      align="bottom"
      items={{
        poll: {
          align: "center",
          label: "Polls",
          component: PollButtonWithSidePanel,
        },
      }}
    />
  );
};

export default CustomBottomBar;
```

### Step 2: Integrar na Customization

```jsx
// customization/index.tsx
import { customize } from "customization-api";
import CustomBottomBar, { POLL_SIDEBAR_NAME } from "./CustomBottomToolBar";
import PollSidebar from "./PollSidebar";

const userCustomization = customize({
  components: {
    videoCall: {
      bottomToolBar: CustomBottomBar,
      customSidePanel: () => {
        return [
          {
            name: POLL_SIDEBAR_NAME,
            component: PollSidebar,
            title: "Polls",
            onClose: () => {},
          },
        ];
      },
    },
  },
});

export default userCustomization;
```

---

## Criar Poll (Form)

### PollSidebar Component

```jsx
// customization/PollSidebar.tsx
import React, { useState } from "react";
import { PrimaryButton, TextInput, $config } from "customization-api";

const PollSidebar = () => {
  const [question, setQuestion] = useState("");
  const [options, setOptions] = useState(["", "", ""]);

  const handleAddOption = () => {
    setOptions([...options, ""]);
  };

  const handleOptionChange = (index, value) => {
    const newOptions = [...options];
    newOptions[index] = value;
    setOptions(newOptions);
  };

  const handleCreatePoll = () => {
    // TODO: Enviar poll via custom event
    console.log("Poll criada:", { question, options });
  };

  return (
    <div style={{ padding: "20px" }}>
      <h3>Criar Votação</h3>

      <TextInput
        placeholder="Pergunta"
        value={question}
        onChangeText={setQuestion}
        style={{ marginBottom: "10px" }}
      />

      {options.map((opt, idx) => (
        <TextInput
          key={idx}
          placeholder={`Opção ${idx + 1}`}
          value={opt}
          onChangeText={(val) => handleOptionChange(idx, val)}
          style={{ marginBottom: "10px" }}
        />
      ))}

      <PrimaryButton
        text="+ Adicionar Opção"
        onPress={handleAddOption}
        containerStyle={{ marginBottom: "10px" }}
      />

      <PrimaryButton
        text="Criar Votação"
        onPress={handleCreatePoll}
      />
    </div>
  );
};

export default PollSidebar;
```

---

## Enviar Poll (Custom Events)

### Broadcasting para Todos

```jsx
import { customEvents, PersistanceLevel } from "customization-api";

const sendPoll = (pollData) => {
  customEvents.send(
    "POLLS",
    JSON.stringify({
      state: { [pollData.id]: pollData },
      action: "SEND_POLL",
      pollId: pollData.id,
    }),
    PersistanceLevel.Channel  // Persiste no canal
  );
};
```

**Importante:** `PersistanceLevel.Channel` garante que novos participantes que entram depois recebem a poll.

---

## Receber Poll (Attendee Side)

### Subscribe ao Evento POLLS

```jsx
import { customEvents } from "customization-api";
import { useEffect, useState } from "react";

const usePollListener = (onPollReceived) => {
  useEffect(() => {
    customEvents.on("POLLS", (args) => {
      const { payload } = args;
      const data = JSON.parse(payload);
      const { action, state, pollId } = data;

      if (action === "SEND_POLL") {
        onPollReceived(state, pollId);
      }
    });

    return () => {
      customEvents.off("POLLS");
    };
  }, [onPollReceived]);
};

export default usePollListener;
```

---

## Votar na Poll

### Submeter Voto

```jsx
const handleVote = (pollId, optionValue) => {
  // 1. Atualiza state local
  setSelectedOption(optionValue);

  // 2. Envia resposta
  customEvents.send(
    "POLL-RESPONSE",
    JSON.stringify({
      pollId,
      responses: [optionValue],
      uid: localUid,
      timestamp: Date.now(),
    }),
    PersistanceLevel.None  // NÃO persiste (é resposta individual)
  );
};
```

---

## Host Recebe Votos & Atualiza Resultado

### Subscribe POLL-RESPONSE

```jsx
useEffect(() => {
  customEvents.on("POLL-RESPONSE", (args) => {
    const { payload } = args;
    const data = JSON.parse(payload);
    const { pollId, responses, uid, timestamp } = data;

    // Adiciona voto à poll
    updatePollWithVote(pollId, responses, uid, timestamp);
  });

  return () => {
    customEvents.off("POLL-RESPONSE");
  };
}, []);
```

### Helper: Adicionar Voto & Recalcular %

```jsx
function addVote(pollId, responses, uid, timestamp) {
  const poll = polls[pollId];
  
  poll.options = poll.options.map((option) => {
    const alreadyVoted = option.votes.some((v) => v.uid === uid);
    
    if (responses.includes(option.value) && !alreadyVoted) {
      return {
        ...option,
        votes: [...option.votes, { uid, timestamp }],
      };
    }
    return option;
  });

  // Recalcular percentuais
  const totalVotes = poll.options.reduce((sum, opt) => sum + opt.votes.length, 0);
  poll.options = poll.options.map((option) => ({
    ...option,
    percent: totalVotes > 0 ? ((option.votes.length / totalVotes) * 100).toFixed(0) : "0",
  }));

  return poll;
}
```

---

## Mostrar Resultados

### UI de Resultados

```jsx
const PollResults = ({ poll }) => {
  return (
    <div>
      <h3>{poll.question}</h3>
      {poll.options.map((option) => (
        <div key={option.value} style={{ marginBottom: "10px" }}>
          <div style={{ display: "flex", justifyContent: "space-between" }}>
            <span>{option.text}</span>
            <span>{option.percent}%</span>
          </div>
          <div style={{
            width: "100%",
            height: "20px",
            background: "#e0e0e0",
            borderRadius: "4px",
            overflow: "hidden",
          }}>
            <div style={{
              width: `${option.percent}%`,
              height: "100%",
              background: "#0066CC",
              transition: "width 0.3s",
            }} />
          </div>
          <small>{option.votes.length} votos</small>
        </div>
      ))}
    </div>
  );
};
```

---

## Fechar Poll

### Host Finaliza

```jsx
const closePoll = (pollId) => {
  customEvents.send(
    "POLLS",
    JSON.stringify({
      action: "CLOSE_POLL",
      pollId,
    }),
    PersistanceLevel.Channel
  );
};
```

---

## Limitações Conhecidas

1. **Sem suporte nativo para pesos** — Agora armazena `uid + timestamp`, não "peso da votação"
   - **Solução MKVote:** Recalcular resultado no backend (multiplicar voto × peso)

2. **Sem criptografia E-2EE nativa** — Votos trafegam em texto
   - **Solução MKVote:** Criptografar antes de enviar via custom event

3. **Sem fingerprinting** — Não garante integridade do voto
   - **Solução MKVote:** Adicionar SHA256 hash do voto

4. **Persistência limitada** — Dados persdem após ~2 minutos se channel vazio
   - **Solução MKVote:** Persistir votos no MongoDB imediatamente

---

## Recursos

- [Polling Guide Oficial](https://appbuilder-docs.agora.io/customization-api/api-examples/customization-polling)
- [Custom Events API](https://appbuilder-docs.agora.io/customization-api/api-reference/custom-events-library)
