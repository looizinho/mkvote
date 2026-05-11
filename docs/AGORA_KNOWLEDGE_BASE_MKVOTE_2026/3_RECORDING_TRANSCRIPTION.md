# Recording & Transcription - Agora.io

## Overview

Agora oferece **Cloud Recording** (grava chamadas na nuvem) e **Transcription** (converte áudio em texto). Ambos são serviços complementares que precisam ser ativados e configurados server-side.

**Recording:**
- Grava áudio + vídeo da chamada
- Armazena na cloud Agora (por padrão)
- Pode fazer download depois
- Formatos: MP4, WebM, etc

**Transcription:**
- Converte áudio em texto (português suportado)
- Pode ser ativada durante/após recording
- Retorna texto com timestamps
- Pode filtrar palavras ofensivas (parcialmente)

---

## Cloud Recording

### Como Funciona

1. **Ativa recording** via API ou dashboard
2. **Agora grava** a chamada (audio + video)
3. **Processa** após chamada terminar (~5-30 min)
4. **Armazena** na cloud ou no seu storage (S3, etc)
5. **Você faz download** com link de acesso

### Ativar Recording

#### Opção A: Via Dashboard Agora
