# Saúde Rápida

<div align="center">

![Next.js](https://img.shields.io/badge/Next.js-16.4.6-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React-19.3-087EA4?style=for-the-badge&logo=react&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-Strict-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-4-38B2AC?style=for-the-badge&logo=tailwindcss&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-Free-3FCF8E?style=for-the-badge&logo=supabase&logoColor=white)

**Encontre a unidade de saúde que pode atendê-lo mais rápido.**

O Saúde Rápida combina tempo de deslocamento, trânsito e relatos de espera
para mostrar qual unidade oferece o menor tempo total.

</div>

---

## Sobre o projeto

O projeto funciona como um “Waze das unidades de saúde”: o usuário informa onde
está e visualiza hospitais, UPAs e postos próximos no mapa e em uma lista.

Cada unidade apresenta:

- tempo estimado de espera;
- tempo de viagem;
- distância aproximada;
- tempo total de deslocamento mais atendimento;
- indicação de quando o tempo de viagem foi estimado.

Os resultados são ordenados do menor para o maior tempo total.

---

## Como funciona

```text
Usuário
   │
   ├── envia a localização
   │
   v
Front-end Next.js
   │
   ├── GET /api/unidades
   └── POST /api/relatos
   │
   v
Route Handlers do Next.js
   │
   ├── Supabase: unidades e relatos
   ├── TomTom: tempo de viagem com trânsito
   └── Haversine: estimativa quando o trânsito não está disponível
```

### Regras principais

| Recurso | Regra |
|---|---|
| Espera exibida | Mediana dos relatos válidos das últimas 3 horas |
| Relato válido | Número inteiro entre 1 e 720 minutos |
| Limite por usuário | Um relato por unidade a cada 30 minutos |
| Identificação | UUID anônimo armazenado no navegador |
| Cache das rotas | 5 minutos por unidade e região da origem |
| Tempo total | Tempo de viagem + tempo de espera |
| Sem relatos | `espera_min = null`, nunca zero |

---

## Tecnologias

| Camada | Tecnologia |
|---|---|
| Interface | Next.js 16, React 19 e Tailwind CSS 4 |
| Linguagem | TypeScript em modo estrito |
| Mapa | Leaflet, React Leaflet e tiles do OpenStreetMap |
| Rotas | TomTom Routing API |
| Banco de dados | Supabase/Postgres |
| Hospedagem sugerida | Vercel Hobby |
| Testes | Vitest |

> O cadastro usa `device_id` anônimo no navegador. Não há login, app nativo,
> notificações ou previsão de espera por horário.

---

## Funcionalidades

- [x] Localização do usuário com fallback para o centro de Sorocaba
- [x] Lista responsiva de unidades ordenada pelo tempo total
- [x] Mapa com marcadores e popups
- [x] Carrossel móvel com navegação por toque
- [x] Registro de tempo de espera
- [x] Mediana resistente a relatos extremos
- [x] Limite de um relato por dispositivo e unidade
- [x] Cache de viagens
- [x] Fallback para estimativa geográfica
- [x] Modo mock para desenvolvimento
- [x] Testes unitários das regras principais

---

## Executando o projeto

### Pré-requisitos

- Node.js
- Conta gratuita no Supabase
- Chave da TomTom, opcional para o modo mock

### Instalação

```bash
npm install
```

### Configuração

Crie um arquivo `.env.local` baseado em `.env.example`:

```env
TOMTOM_API_KEY=
SUPABASE_URL=
SUPABASE_SERVICE_ROLE_KEY=
NEXT_PUBLIC_USE_MOCK=true
```

Para usar dados simulados:

```env
NEXT_PUBLIC_USE_MOCK=true
```

Para usar a API real:

```env
NEXT_PUBLIC_USE_MOCK=false
```

> Nunca exponha `SUPABASE_SERVICE_ROLE_KEY` ou `TOMTOM_API_KEY` no front-end.
> Essas informações devem permanecer apenas no servidor.

### Desenvolvimento

```bash
npm run dev
```

Acesse `http://localhost:3000`.

---

## Scripts disponíveis

| Comando | Finalidade |
|---|---|
| `npm run dev` | Inicia o servidor de desenvolvimento |
| `npm run build` | Gera o build de produção |
| `npm start` | Executa o build de produção |
| `npm run lint` | Executa o ESLint |
| `npm test` | Executa todos os testes uma vez |
| `npm run test:watch` | Executa os testes em modo observação |

---

## Estrutura do projeto

```text
app/
├── api/
│   ├── relatos/       # Registro de novos relatos
│   └── unidades/      # Lista, espera e tempo de viagem
├── globals.css        # Estilos globais e animações
├── layout.tsx         # Layout raiz e metadados
└── page.tsx           # Tela principal

components/            # Mapa, cards e formulário
hooks/                 # Geolocalização
lib/                   # Regras, integrações e funções puras
types/                 # Tipos e constantes do contrato
docs/                  # Contrato e documentação do código
tests/                 # Testes unitários
public/                # Ícones e recursos estáticos
```

---

## Contrato e documentação

Os detalhes completos das respostas da API estão em:

- [Contrato da API](docs/contrato.md)
- [Índice da documentação do código](docs/INDICE_DOCUMENTACAO.md)
- [Tipos compartilhados](types/unidade.ts)

Qualquer alteração no contrato precisa atualizar juntos
`docs/contrato.md` e `types/unidade.ts`.

---

## Qualidade e testes

Os testes cobrem:

- cálculo da mediana;
- janela de três horas;
- limites de tempo;
- coordenadas geográficas;
- fallback de viagem;
- integração simulada da TomTom;
- respostas do cliente HTTP;
- dados do modo mock.

Execute:

```bash
npm test
```

---

## Degradação elegante

Se TomTom estiver sem chave, fora do ar ou sem cota, o sistema continua
funcionando com uma estimativa baseada em distância em linha reta,
fator de desvio e velocidade média urbana.

Nesse caso, a interface marca a unidade com o selo **estimado**.

---

## Segurança e privacidade

- Nenhuma chave privada é enviada ao navegador.
- O identificador do dispositivo é anônimo.
- A service role do Supabase é usada somente no servidor.
- Relatos são validados duas vezes: no navegador e no servidor.
- Dados locais não substituem a validação da API.

---

## Contribuindo

1. Crie uma branch a partir de `main`.
2. Faça alterações pequenas e coerentes.
3. Siga o padrão de commits `tipo(escopo): descrição`.
4. Execute testes e lint antes do Pull Request.
5. Nunca faça push direto em `main`.

Exemplos de commit:

```text
feat(front): adicionar carrossel móvel
fix(api): validar coordenadas fora do intervalo
docs(readme): melhorar apresentação do projeto
test(geo): cobrir coordenadas inválidas
```
