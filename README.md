# <img src="public/futura-f.png" width="35" height="65" style="margin-right:3px;"/>utura EaD

<!-- A imagem acima substitui a letra "F" do nome. Salve o arquivo da letra em public/futura-f.png
     (ou troque o caminho no src). Ajuste width e height se a imagem tiver outra proporção. -->

Futura EaD é uma plataforma de ensino a distância em que o aluno se matricula em cursos, assiste às aulas no próprio ritmo, faz atividades corrigidas automaticamente e recebe um certificado em PDF ao concluir.
Os professores criam os cursos, organizam o conteúdo em módulos e aulas e cadastram as atividades. Depois disso, a plataforma funciona sozinha, 24 horas por dia.

O mascote da plataforma é o **Otto**, um polvo que acompanha o aluno em toda a jornada: dá as boas-vindas, comemora cada aula concluída e entrega o certificado no final.

## Objetivo

Oferecer uma forma simples e acessível de estudar a distância, com conteúdo organizado, acompanhamento do progresso e certificação automática para quem atinge os requisitos do curso.

## Objetivos de Desenvolvimento Sustentável (ODS)

O Futura EaD está alinhado aos seguintes Objetivos de Desenvolvimento Sustentável da ONU:

* **ODS 4 – Educação de Qualidade:** amplia o acesso a cursos com conteúdo estruturado, avaliação e certificação.
* **ODS 8 – Trabalho Decente e Crescimento Econômico:** permite que as pessoas desenvolvam habilidades profissionais no próprio tempo.
* **ODS 10 – Redução das Desigualdades:** o aluno estuda de onde estiver, sem depender de deslocamento ou de horários fixos.

## Como Rodar o Projeto

1. Abra o terminal na pasta raiz do repositório e instale as dependências:

```bash
npm install
```

2. Crie um arquivo `.env.local` na raiz do projeto e adicione as suas credenciais do Supabase:

```env
NEXT_PUBLIC_SUPABASE_URL=sua_url_do_supabase_aqui
NEXT_PUBLIC_SUPABASE_ANON_KEY=sua_chave_anon_do_supabase_aqui
```

3. No painel do Supabase, abra o **SQL Editor** e execute os arquivos da pasta `supabase/migrations` para criar as tabelas.

4. Inicie o servidor de desenvolvimento:

```bash
npm run dev
```

Acesse `http://localhost:3000` no seu navegador para visualizar.

## Funcionalidades

### Autenticação e Usuários

* Cadastro de usuários
* Login seguro com Supabase Auth
* Controle de acesso por papel: **Aluno** e **Professor**

### Catálogo de Cursos

* Página inicial com os cursos em destaque
* Página de cada curso com descrição, carga horária, módulos e aulas
* Matrícula do aluno no curso

### Aulas

* Player de vídeo e aulas em texto
* Lista de aulas do módulo ao lado do conteúdo
* Botão para marcar a aula como concluída

### Atividades

* Atividades de múltipla escolha ao fim de cada módulo
* Correção automática, com a nota mostrada na hora
* Comparação com a nota mínima exigida pela atividade

### Progresso e Certificação

* Barra de progresso de cada curso em **Meus cursos**
* Histórico de aulas concluídas e de tentativas nas atividades
* Emissão automática do certificado em PDF quando o aluno conclui todas as aulas e atinge a nota mínima em todas as atividades

### Painel do Professor

* Criação de cursos, módulos e aulas
* Upload de vídeos
* Cadastro de atividades, questões e alternativas
* Lista de alunos matriculados

### Pagamentos

* Compra do curso com aprovação simulada
* A venda aprovada gera o evento `VendaAprovada`, que libera a matrícula automaticamente

## Arquitetura

A plataforma segue uma arquitetura de **microsserviços orientada a eventos**, organizada em camadas:

| Camada | Responsabilidade | Tecnologia |
|---|---|---|
| 1. Apresentação | Interface usada por alunos e professores | Next.js, React, Tailwind CSS, TypeScript |
| 2. API Gateway | Porta de entrada única das requisições | Next.js API Routes / Server Actions |
| 3. Microsserviços | Regras de negócio separadas por domínio | Node.js / TypeScript |
| 4. Mensageria | Comunicação assíncrona entre serviços | Barramento de eventos (RabbitMQ previsto) |
| 5. Persistência | Banco de dados, autenticação e cache | Supabase (PostgreSQL e Auth), Redis previsto |
| 6. Mídia | Armazenamento e entrega dos vídeos | Supabase Storage, FFmpeg (HLS), Cloudflare CDN |

### Microsserviços

Cada serviço fica em uma pasta própria dentro de `src/services` e pode ser separado em um microsserviço independente:

* **Auth:** cadastro, login e papéis de acesso
* **Catálogo:** cursos, módulos, aulas e trilhas
* **Progresso:** aulas concluídas, notas e certificados
* **Mídia:** upload e entrega dos vídeos
* **Comunicação:** avisos e notificações aos alunos
* **Pagamentos:** compras e liberação dos cursos

### Estrutura de Pastas

```
futura-ead/
├── src/
│   ├── app/              # Páginas do site e rotas da API
│   ├── services/         # Microsserviços (auth, catalogo, progresso, midia, comunicacao, pagamentos)
│   ├── events/           # Barramento de eventos
│   ├── lib/              # Conexão com Supabase e funções auxiliares
│   ├── components/       # Componentes reutilizáveis (Otto, cartões, cabeçalho…)
│   └── types/            # Tipos TypeScript das tabelas
├── supabase/migrations/  # Scripts SQL do banco de dados
├── docs/                 # Diagramas e relatório
└── public/               # Imagens e ícones
```

## Modelo de Dados

### Entidades

* **Curso:** nome, descrição e carga horária
* **Módulo:** divide o curso em partes, com nome e ordem
* **Aula:** título, tipo de mídia, URL do vídeo e texto
* **Usuário:** nome, e-mail, senha e papel (aluno ou professor)
* **Matrícula:** liga o aluno ao curso, com data de início e status (Ativo, Concluído, Cancelado)
* **Atividade:** bloco de exercícios do módulo, com título e nota mínima
* **Questão:** enunciado e tipo de questão
* **Alternativa:** texto e indicação de alternativa correta
* **Progresso_Aula:** registra as aulas concluídas pelo aluno
* **Progresso_Atividade:** registra a nota obtida e a data de cada tentativa
* **Certificação:** data de emissão e link do PDF do certificado

### Relacionamentos

| Entidade Origem | Entidade Destino | Tipo de Relacionamento | Descrição |
|---|---|---|---|
| Curso | Módulo | 1 : N | Um curso possui vários módulos; um módulo pertence a apenas um curso. |
| Módulo | Aula | 1 : N | Um módulo possui várias aulas; uma aula pertence a apenas um módulo. |
| Módulo | Atividade | 1 : N | Um módulo possui várias atividades; uma atividade pertence a apenas um módulo. |
| Atividade | Questão | 1 : N | Uma atividade possui várias questões; uma questão pertence a apenas uma atividade. |
| Questão | Alternativa | 1 : N | Uma questão possui várias alternativas; uma alternativa pertence a apenas uma questão. |
| Usuário | Matrícula | 1 : N | Um usuário pode ter várias matrículas; uma matrícula pertence a apenas um usuário. |
| Curso | Matrícula | 1 : N | Um curso pode ter várias matrículas; uma matrícula se refere a apenas um curso. |
| Usuário | Curso | N : N | Um aluno pode se matricular em vários cursos e um curso pode ter vários alunos, por meio da Matrícula. |
| Usuário | Progresso_Aula | 1 : N | Um usuário registra o progresso de várias aulas; cada registro pertence a apenas um usuário. |
| Aula | Progresso_Aula | 1 : N | Uma aula pode ser concluída por vários usuários; cada registro se refere a apenas uma aula. |
| Usuário | Progresso_Atividade | 1 : N | Um usuário pode ter várias tentativas em atividades; cada tentativa pertence a apenas um usuário. |
| Atividade | Progresso_Atividade | 1 : N | Uma atividade pode ter tentativas de vários usuários; cada tentativa se refere a apenas uma atividade. |
| Usuário | Certificação | 1 : N | Um usuário pode receber vários certificados; cada certificado pertence a apenas um usuário. |
| Curso | Certificação | 1 : N | Um curso pode gerar vários certificados; cada certificado se refere a apenas um curso. |

## Tecnologias Utilizadas

<p>
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/nextjs/nextjs-original.svg" alt="Next.js" title="Next.js" width="40" height="40"/>&nbsp;&nbsp;&nbsp;
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/react/react-original.svg" alt="React" title="React" width="40" height="40"/>&nbsp;&nbsp;&nbsp;
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/typescript/typescript-original.svg" alt="TypeScript" title="TypeScript" width="40" height="40"/>&nbsp;&nbsp;&nbsp;
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/tailwindcss/tailwindcss-original.svg" alt="Tailwind CSS" title="Tailwind CSS" width="40" height="40"/>&nbsp;&nbsp;&nbsp;
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/supabase/supabase-original.svg" alt="Supabase" title="Supabase" width="40" height="40"/>&nbsp;&nbsp;&nbsp;
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/postgresql/postgresql-original.svg" alt="PostgreSQL" title="PostgreSQL" width="40" height="40"/>&nbsp;&nbsp;&nbsp;
  <img src="https://cdn.simpleicons.org/lucide" alt="Lucide" title="Lucide" width="40" height="40"/>&nbsp;&nbsp;&nbsp;
  <img src="https://cdn.simpleicons.org/vercel/8B949E" alt="Vercel" title="Vercel" width="40" height="40"/>
</p>

### Front-end

* **Next.js:** projeto organizado, rotas e melhor desempenho
* **React:** criação de componentes reutilizáveis
* **TypeScript:** desenvolvimento do site evitando erros
* **Tailwind CSS:** interface moderna e responsiva

### Banco de Dados e Backend

* **Supabase:** autenticação, banco de dados PostgreSQL e armazenamento dos vídeos
* **Next.js API Routes:** API Gateway da plataforma

### Hospedagem

* **Vercel:** hospedagem do site

### Bibliotecas

* **Supabase JS** e **Supabase SSR:** conexão com o banco de dados e com a autenticação
* **pdf-lib:** geração dos certificados em PDF
* **Lucide React:** ícones do site

## Público-Alvo

* Alunos que querem estudar a distância
* Professores que querem publicar cursos
* Instituições de ensino

## Principais Módulos

* Login e Cadastro
* Catálogo de Cursos
* Página do Curso
* Meus Cursos
* Player de Aulas
* Atividades
* Certificados
* Painel do Professor

## Equipe

<!-- Complete com os integrantes do grupo, ou deixe só o seu nome se o projeto for individual. -->

| Integrante | Função | GitHub |
|---|---|---|
| **Julia Santos** | Desenvolvedora Full Stack | [@JuliaKarollyne-O-Santos](https://github.com/JuliaKarollyne-O-Santos) |

- **Orientador: [nome do professor]**

## Projeto Acadêmico

Projeto acadêmico desenvolvido na **Fatec Taubaté (Faculdade de Tecnologia de Taubaté)**, na disciplina de **[nome da disciplina]**, para fins de estudo e avaliação.

O projeto foi desenvolvido para aplicação de:

* Desenvolvimento Web
* Banco de Dados (MER/DER)
* Arquitetura de Microsserviços
* Arquitetura Orientada a Eventos
* Streaming de Vídeo
* Interface Responsiva