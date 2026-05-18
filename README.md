# Sistema de checklist de qualidade

> Sistema de gestão de qualidade para empresa de APH (Atendimento Pré-Hospitalar) privado.

---
<img width="1920" height="1080" alt="PORTFÓLIO - SISTEMA CRONOS" src="https://github.com/user-attachments/assets/81c0c949-8fa5-4bab-b480-674baeb400e7" />


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

<img width="1920" height="1080" alt="PORTFÓLIO - SISTEMA CRONOS" src="https://github.com/user-attachments/assets/81c0c949-8fa5-4bab-b480-674baeb400e7" />

### Gestor (desktop)

<img width="1920" height="1080" alt="2" src="https://github.com/user-attachments/assets/845349a8-d5ce-4965-b997-760966e18813" />
<img width="1920" height="1080" alt="3" src="https://github.com/user-attachments/assets/30779b72-885b-4836-9750-8255867cee82" />
<img width="1920" height="1080" alt="4" src="https://github.com/user-attachments/assets/a9ce3276-85fb-4eae-9c2e-99c1efff6b53" />
<img width="1920" height="1080" alt="5" src="https://github.com/user-attachments/assets/e641e5bc-74ac-4dce-95df-bac636740a08" />
<img width="1920" height="1080" alt="6" src="https://github.com/user-attachments/assets/ece91f37-d7c9-47e2-b279-b9d9eebe6a76" />


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
