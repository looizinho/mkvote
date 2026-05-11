# Agora App Builder

## Overview

Agora App Builder é uma solução **pré-construída** que permite criar aplicações de videoconferência sem começar do zero. É baseada em React e oferece UI pronta, com capacidade de customização via Customization API.

**Use quando:**
- Precisa de videoconferência rápido
- UI padrão é aceitável
- Quer customizar com React

**NÃO use quando:**
- Precisa de UI completamente diferente
- Constraints de performance rigorosas

---

## Setup & Installation

### Pré-requisitos
- Node.js 16+ 
- npm ou yarn
- Agora App ID (free tier disponível em agora.io)

### Instalação Básica

```bash
# Clone o template
git clone https://github.com/AgoraIO-Community/web-sdk-samples
cd web-sdk-samples/app-builder-on-the-web

# Install dependencies
npm install

# Configure .env
REACT_APP_AGORA_APP_ID=seu_app_id_aqui

# Run
npm start
```

**Output:** Aplicação roda em `http://localhost:3000`

---

## Customization API

### O que é?

A **Customization API** permite modificar:
- Bottom toolbar (botões, ações)
- Side panels (painéis laterais)
- Custom components
- Styling (cores, fontes)
- Event listeners

### Fluxo Básico

```jsx
// customization/index.tsx
import { customize } from "customization-api";

const userCustomization = customize({
  components: {
    videoCall: {
      bottomToolBar: CustomBottomBar,
      customSidePanel: () => [
        { name: "poll-panel", component: PollPanel, title: "Polls" }
      ]
    }
  }
});

export default userCustomization;
```

### Componentes Disponíveis

| Componente | Descrição |
|-----------|-----------|
| `bottomToolBar` | Toolbar na base (botões de ação) |
| `customSidePanel` | Painéis customizados (lado direito) |
| `videoCall` | Container principal da chamada |

### Exemplo: Adicionar Botão Customizado

```jsx
import { ToolbarPreset, TertiaryButton, ToolbarItem } from "customization-api";

const CustomButton = () => (
  <ToolbarItem>
    <TertiaryButton
      text="Meu Botão"
      onPress={() => console.log("Clicado!")}
    />
  </ToolbarItem>
);

const CustomToolBar = () => (
  <ToolbarPreset
    align="bottom"
    items={{ custom: { component: CustomButton } }}
  />
);
```

---

## Componentes Disponíveis (UI Library)

App Builder expõe uma UI library com componentes prontos:

- `PrimaryButton` — Botão principal (ação importante)
- `TertiaryButton` — Botão secundário
- `TextInput` — Campo de texto
- `ToolbarPreset` — Toolbar customizável
- `useSidePanel` — Hook para controlar side panels

---

## Performance & Limitations

### Limitações Conhecidas

1. **Browser Support:** Chrome 80+, Firefox 67+, Safari 12+
2. **Video Streams:** Máx ~20 participantes simultâneos (depende da máquina)
3. **Mobile:** Suporta iOS (Safari) e Android (Chrome), mas com degradação
4. **Recording:** Precisa de configuração server-side

### Performance Tips

- Use `videoProfile` para ajustar qualidade
- Desabilite vídeo pra aumentar áudio quality se necessário
- Limite participants se for presencial + online

---

## Licensing & Pricing

- **Free Tier:** 10,000 minutos/mês (gratuito)
- **Paid:** $0.99 - $3.99 por 1000 minutos (depende de features)
- **Incluso na Free:** Video, audio, screen share, recording (básico)

---

## Integração com MKVote

### O que usaremos

✅ App Builder UI (base da videoconferência)
✅ Customization API (adicionar polling, toolbar customizado)
✅ Recording (com nossa persistência por cima)
✅ Custom Events (sincronização de votos)

### O que NÃO usaremos

❌ Storage nativo do Agora (usaremos MongoDB)
❌ User management do Agora (usaremos CPF/CNPJ)
❌ Recording storage do Agora (baixaremos e persistiremos)

---

## Recursos Úteis

- [Documentação Oficial](https://docs.agora.io/en/app-builder/overview)
- [GitHub - App Builder](https://github.com/AgoraIO-Community/app-builder-core)
- [Customization API Docs](https://appbuilder-docs.agora.io/customization-api/quickstart)
