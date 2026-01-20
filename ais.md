# 🤖 Análise de Modelos de IA — PrimeForm

Este documento apresenta uma análise comparativa dos modelos de IA disponíveis no mercado e recomendações para o PrimeForm.

---

## Requisitos do PrimeForm

O modelo de IA ideal deve atender aos seguintes requisitos:

1. **Capacidade Multimodal (Visão)** — Análise de fotos de alimentos e progresso corporal
2. **Raciocínio Adaptativo** — Ajustar treinos com base em humor, energia e tempo disponível
3. **Qualidade em Português Brasileiro** — Respostas naturais em pt-BR
4. **Baixa Latência** — Experiência fluida no app mobile
5. **Custo Operacional Baixo** — Viabilidade financeira em escala

---

## Comparativo de Modelos

| Modelo | Provedor | Visão | Input (1M tokens) | Output (1M tokens) | Latência | Contexto |
|--------|----------|-------|-------------------|--------------------|-----------| ---------|
| **Groq Llama 4 Scout** | Groq | ✅ | ~$0.11 | ~$0.34 | ⚡ Ultra-rápido | 128K |
| **GPT-4o-mini** | OpenAI | ✅ | $0.15 | $0.60 | Rápido | 128K |
| **GPT-4o** | OpenAI | ✅ | $2.50 | $10.00 | Médio | 128K |
| **Gemini 2.0 Flash** | Google | ✅ | $0.10 | $0.40 | ⚡ Rápido | 1M+ |
| **Claude 3.5 Haiku** | Anthropic | ✅ | $0.25 | $1.25 | Rápido | 200K |
| **Claude 3.5 Sonnet** | Anthropic | ✅ | $3.00 | $15.00 | Médio | 200K |

---

## Análise por Modelo

### Groq Llama 4 Scout (Atual)

**Status:** Implementado no projeto

**Prós:**
- Latência mais baixa do mercado (resposta instantânea)
- Custo muito competitivo
- Funciona bem para análise de imagens
- API compatível com OpenAI

**Contras:**
- Qualidade de raciocínio inferior ao GPT-4o/Claude
- Menos consistente em respostas estruturadas (JSON)
- Português pode ser menos natural

**Ideal para:** MVP, alta demanda, fallback

---

### GPT-4o-mini (OpenAI)

**Prós:**
- Excelente custo-benefício
- Raciocínio superior para tarefas complexas
- Muito consistente em JSON/estruturado
- Ótimo em português

**Contras:**
- Latência maior que Groq
- Custo ligeiramente maior que Gemini Flash

**Ideal para:** Análise de imagens, interações que exigem precisão

---

### Gemini 2.0 Flash (Google)

**Prós:**
- Preço mais agressivo do mercado (~$0.10/1M input)
- Janela de contexto enorme (1M+ tokens)
- Multimodal nativo (fotos/vídeos)
- Tier gratuito generoso no Google Cloud
- Bom em português brasileiro

**Contras:**
- Integração com Google Cloud pode ser mais complexa
- Menos "personalidade" que Claude

**Ideal para:** Custo-benefício, histórico longo do usuário, análises em escala

---

### Claude 3.5 Haiku (Anthropic)

**Prós:**
- Tom mais humano e empático
- Excelente para contextos emocionais
- Respostas menos robóticas
- Muito bom em português

**Contras:**
- Ligeiramente mais caro que GPT-4o-mini
- API da Anthropic menos difundida

**Ideal para:** Interações de humor/energia, mensagens motivacionais, "companheiro de decisões"

---

### GPT-4o / Claude 3.5 Sonnet (Premium)

**Prós:**
- Máxima qualidade de raciocínio
- Análises mais precisas e detalhadas

**Contras:**
- Custo 10-20x maior que modelos leves
- Não justifica para uso diário no PrimeForm

**Ideal para:** Funcionalidades premium futuras, análises complexas pontuais

---

## Estimativa de Custo por Usuário

### Cenário de Uso Mensal Típico

| Atividade | Frequência | Tokens Estimados |
|-----------|------------|------------------|
| Análise de refeições (foto) | 30x | ~45.000 |
| Análise de progresso corporal | 4x | ~6.000 |
| Ajustes de treino (texto) | 60x | ~18.000 |
| Check-ins de humor/energia | 30x | ~9.000 |
| **Total** | | **~78.000 tokens** |

### Custo Mensal por Usuário

| Modelo | USD/mês | BRL/mês* |
|--------|---------|----------|
| **Gemini 2.0 Flash** | $0.01 - $0.03 | R$ 0,06 - R$ 0,18 |
| **GPT-4o-mini** | $0.02 - $0.05 | R$ 0,12 - R$ 0,30 |
| **Groq Llama 4 Scout** | $0.02 - $0.04 | R$ 0,12 - R$ 0,24 |
| **Claude 3.5 Haiku** | $0.03 - $0.08 | R$ 0,18 - R$ 0,48 |
| **GPT-4o** | $0.30 - $0.50 | R$ 1,80 - R$ 3,00 |

*\*Considerando USD 1 = BRL 6*

---

## Recomendação

### Estratégia Híbrida (Recomendada)

```
┌─────────────────────────────────────────────────────────────┐
│                     PRIMEFORM AI STACK                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  📸 ANÁLISE DE IMAGENS (food/body)                          │
│  └── Gemini 2.0 Flash ou GPT-4o-mini                        │
│      • Multimodal barato e preciso                          │
│                                                             │
│  💬 INTERAÇÕES DIÁRIAS (chat, ajustes)                      │
│  └── Claude 3.5 Haiku                                       │
│      • Tom empático para estado emocional                   │
│      • Respostas que parecem de um "amigo fitness"          │
│                                                             │
│  ⚡ FALLBACK / ALTA DEMANDA                                  │
│  └── Groq Llama (atual)                                     │
│      • Latência ultra-baixa                                 │
│      • Custo mínimo em picos                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Modelo Único (Se necessário escolher apenas um)

**🥇 Gemini 2.0 Flash**

Motivos:
1. Preço ~50% menor que GPT-4o-mini
2. Contexto de 1M tokens = app pode "lembrar" meses de histórico
3. Multimodal nativo (fotos sem custo extra)
4. Excelente em português brasileiro
5. Google Cloud oferece tier gratuito generoso

---

## Viabilidade Financeira

Com custo de IA entre **R$ 0,10 e R$ 0,50 por usuário/mês**:

| Plano | Preço | Custo IA | Margem Bruta |
|-------|-------|----------|--------------|
| Gratuito | R$ 0 | R$ 0,15 | -R$ 0,15 (lead) |
| Básico | R$ 9,90 | R$ 0,25 | R$ 9,65 (97%) |
| Pro | R$ 19,90 | R$ 0,50 | R$ 19,40 (97%) |

**Conclusão:** O custo de IA é praticamente irrelevante para o modelo de negócio. Os maiores custos serão:
- Aquisição de usuários (marketing)
- Infraestrutura (Supabase, storage de imagens)
- Desenvolvimento e manutenção

---

## Implementação Atual

O projeto atualmente utiliza:

- **Modelo:** `meta-llama/llama-4-scout-17b-16e-instruct`
- **Provedor:** Groq
- **Arquivo:** `lib/groq.ts`

### Funcionalidades Implementadas

1. `analyzeFoodImage()` — Análise nutricional de fotos de refeições
2. `analyzeBodyImage()` — Estimativa de composição corporal
3. `detectImageType()` — Classificação automática (food/body/unknown)

---

## Próximos Passos

1. [ ] Avaliar migração para Gemini 2.0 Flash (custo-benefício)
2. [ ] Implementar fallback multi-provedor
3. [ ] Adicionar cache de respostas para padrões comuns
4. [ ] Criar sistema de "templates inteligentes" para economizar chamadas
5. [ ] Implementar métricas de uso por usuário

---

## Referências

- [OpenAI Pricing](https://openai.com/pricing)
- [Google AI Pricing](https://ai.google.dev/pricing)
- [Anthropic Pricing](https://www.anthropic.com/pricing)
- [Groq Pricing](https://groq.com/pricing/)

---

*Última atualização: Janeiro 2026*

