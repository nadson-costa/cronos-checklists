# Sistema de checklist de qualidade

> Sistema de gestão de qualidade para empresa de APH (Atendimento Pré-Hospitalar) privado.

---

## O problema

A empresa opera frotas de ambulâncias com médicos e enfermeiros que precisam verificar diariamente dezenas de equipamentos e insumos antes de cada plantão.

O processo era feito com papel: sem rastreabilidade precisa, sem alertas automáticos para a gestão quando houver pendências ou faltas em recursos/insumos, e sem histórico confiável para auditorias.

Na prática isso significava:

- Equipamentos críticos podem ser esquecidos sem registro
- Diretores sem visibilidade do estado real da frota
- Nenhuma evidência documental em caso de incidentes
- Retrabalho no fim do mês para consolidar informações

---

## A solução

O sistema criado digitaliza esse fluxo de ponta a ponta: o operador faz o checklist pelo celular antes do plantão, assina digitalmente, e o gestor acompanha tudo em tempo real pelo dashboard.

### Como funciona

```
Operador                          Gestor
────────                          ──────
Seleciona a VTR          →
Escolhe o tipo de checklist
Preenche cada item               Recebe alerta se algo estiver
(ENC / REP / Status)       →     faltando ou incompleto
Assina digitalmente
Envia                      →     Dashboard atualizado
                                 PDF disponível para auditoria
```

**Regras de negócio principais:**
- Um checklist por viatura por dia
- Status calculado no backend: `ok` / `parcial` / `faltando`
- Itens especiais: controle de carga de cilindro de O₂ (%) e nº de lacre de bolsa de medicamentos
- Justificativa obrigatória quando o checklist não pode ser realizado
- Alertas gerados automaticamente para o gestor em caso de inconformidades

---

## Interface

### Operador (mobile-first)

<!-- screenshot: home do operador -->
> Tela inicial com viatura ativa, status do plantão e botão de iniciar checklist

<!-- screenshot: checklist em preenchimento -->
> Checklist com seções colapsáveis, stepper touch-friendly e toggle de reposição

<!-- screenshot: assinatura -->
> Revisão de inconformidades e pad de assinatura digital antes do envio

### Gestor (dashboard desktop)

<!-- screenshot: dashboard visão geral -->
> KPIs da frota, gráfico de conformidade e tabela com status de todas as viaturas

<!-- screenshot: alertas -->
> Lista de alertas de inconformidades com filtros por viatura, status e data

<!-- screenshot: configuração de checklist -->
> Tabela de itens configuráveis com sidebar de edição — sem deploy necessário para ajustes

---

## Arquitetura

```
┌─────────────────────────────────────────┐
│                  VPS                    │
│                                         │
│   ┌──────────┐       ┌──────────────┐   │
│   │  Nginx   │──────▶│  Next.js 14  │   │
│   │ (proxy)  │       │  (frontend)  │   │
│   └──────────┘       └──────────────┘   │
│        │                                │
│        │             ┌──────────────┐   │
│        └────────────▶│  FastAPI     │   │
│                      │  (backend)   │   │
│                      └──────┬───────┘   │
└─────────────────────────────┼───────────┘
                              │
                    ┌─────────▼────────┐
                    │    Supabase      │
                    │  PostgreSQL +    │
                    │  Auth            │
                    └──────────────────┘
```

**Dois subdomínios, uma aplicação:**
- `app.` → interface mobile para operadores
- `gestor.` → dashboard desktop para diretores

O roteamento por subdomínio é feito no middleware do Next.js com mesma build mas grupos de rotas separados.

---

## Stack

| Camada | Tecnologia |
|---|---|
| Frontend | Next.js 14 (App Router) · TypeScript · Tailwind CSS · shadcn/ui · Zustand |
| Backend | Python 3.12 · FastAPI · Pydantic v2 |
| Banco de dados | Supabase (PostgreSQL + Auth) |
| Infra | Docker · Docker Compose · Nginx · VPS |
| PDF | fpdf2 (geração server-side) |
| Auth | Supabase Auth · JWT · httpOnly cookies |

---

## Funcionalidades

**Operador**
- [x] Login e autenticação segura
- [x] Seleção de viatura e tipo de checklist (básico / avançado)
- [x] Confirmação de flags condicionais (veículo para evento, lacre rompido)
- [x] Checklist com seções colapsáveis e itens touch-friendly
- [x] Controle de reposição com quantidade ou percentual
- [x] Assinatura digital via canvas
- [x] Registro de justificativa quando o checklist não pode ser realizado
- [x] Histórico de checklists e justificativas

**Gestor**
- [x] Dashboard com KPIs em tempo real
- [x] Alertas automáticos de inconformidades
- [x] Gerenciamento de frota (criar, editar, desativar viaturas)
- [x] Gerenciamento de usuários (operadores e diretores)
- [x] Configuração de itens do checklist sem necessidade de deploy
- [x] Download de PDF de qualquer checklist ou justificativa
- [x] Soft delete de checklists (permite resubmissão no mesmo dia)

---

## Decisões técnicas relevantes

**Status calculado 100% no backend**
O frontend nunca decide se um item está `ok`, `parcial` ou `faltando`, isso é responsabilidade exclusiva da API. Garante consistência independente do cliente.

**Alertas gerados atomicamente**
No mesmo ciclo de escrita do checklist, a API gera os alertas de inconformidade. Sem job em background, sem eventual consistency.

**Itens condicionais**
Alguns itens do checklist só aparecem em contextos específicos (ex: equipamentos de evento, lacre da bolsa de medicamentos). A flag `conditional_on` no banco controla isso.


## Status

Aplicação em produção, sendo utilizada pelo cliente.
