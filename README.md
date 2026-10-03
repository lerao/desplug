# Source DesplugAI

| Backend | Frontend |
| --- | --- |
| [![GitHub repo](https://img.shields.io/badge/github-repo-blue?logo=github)](https://github.com/lerao/desplug-backend/) | [![GitHub repo](https://img.shields.io/badge/github-repo-green?logo=github)](https://github.com/lerao/desplug-frontend/) |

Para copiar os projetos, use 
```
git clone --recurse-submodules https://github.com/lerao/desplug.git
```


# Documento de Especificação de Requisitos – Plataforma DesplugAI

## 1. Visão Geral

A *Plataforma DesplugAI* é uma solução web colaborativa e inteligente criada para apoiar professores das redes municipais, estaduais e privadas de educação na criação, adaptação, aplicação e compartilhamento de planos de aula e atividades pedagógicas, com ênfase especial no cumprimento das diretrizes da **BNCC da Computação** (Educação Infantil ao Ensino Médio), além de disciplinas tradicionais do currículo escolar.

Muitas práticas e atividades criativas desenvolvidas no "chão da escola" acabam restritas a iniciativas locais ou a pontuais eventos de divulgação. A *DesplugAI* soluciona essa fragmentação ao fornecer um repositório centralizado de práticas pedagógicas e atividades (especialmente computação desplugada e práticas maker), associadas a habilidades e competências da BNCC.

O grande diferencial da plataforma é seu **Assistente Inteligente Integrado (DesplugAI Studio)**. Com o suporte de Inteligência Artificial Generativa, um professor pode pegar qualquer atividade compartilhada no repositório (ou criar uma do zero) e adaptá-la livremente com base no contexto real da sua sala de aula: informando os materiais disponíveis (ex.: sucata de computadores desktop, papelão, tablets ou nenhum recurso digital), o nível de maturidade e faixa etária da turma, os interesses dos alunos e as necessidades específicas de aprendizagem.

Além do ecossistema docente, a plataforma atende diretamente às **Secretarias de Educação (Municipais e Estaduais)**, fornecendo painéis analíticos e relatórios de acompanhamento de engajamento pedagógico, mapeamento de habilidades trabalhadas nas escolas e gestão dos professores vinculados à sua rede.

---

## 2. Atores do Sistema

Os atores representam os diferentes perfis que integram o quadro de usuários e visitantes da plataforma:

* **Administrador Global:** Responsável pela governança master do sistema, gestão e credenciamento de Secretarias de Educação, aprovação de professores sem vínculo direto com secretarias, moderação de conteúdo e manutenção da base de habilidades da BNCC.
* **Secretaria de Educação:** Representante institucional (gestor pedagógico ou técnico de rede municipal/estadual) que gerencia o quadro de professores de sua jurisdição, homologa seus cadastros e acompanha relatórios e métricas de produção pedagógica.
* **Professor:** Usuário autenticado que cadastra, consulta, favorita e avalia planos de aula. Caso esteja aprovado/homologado, tem acesso irrestrito ao assistente de IA para geração e customização contextual de atividades.
* **Visitante:** Usuário anônimo ou estudante/educador não autenticado que navega pela lista pública de planos de aula e atividades aprovadas no repositório.

---

## 3. Perfis dos Atores

### 1. Perfis de Acesso ao Sistema (Nível Global)

* **Administrador Global (`ROLE_ADMIN`):** Privilégios totais de sistema. Gerencia secretarias, aprova qualquer professor, modera planos de aula, edita tabelas de apoio (BNCC, disciplinas, eixos) e acessa todos os relatórios consolidados.
* **Gestor de Secretaria (`ROLE_SECRETARIA`):** Acesso à área de gestão da rede de ensino. Permite buscar professores por e-mail, homologar seu vínculo com a secretaria e visualizar dashboards analíticos agregados e relatórios periódicos da sua respectiva rede.
* **Professor Homologado (`ROLE_PROFESSOR`):** Professor com cadastro aprovado (por uma Secretaria ou Administrador) que possui e-mail institucional válido (`.br`). Possui acesso completo: criação de planos, adaptação de planos existentes via IA, geração de planos do zero via IA e publicação de relatos de aplicação.
* **Professor Pendente (`ROLE_PROFESSOR_PENDING`):** Professor recém-cadastrado cujo e-mail institucional ainda aguarda homologação. Pode navegar no repositório, favoritar planos e cadastrar planos de forma 100% manual (sem uso do módulo de IA).
* **Usuário Anônimo (`ROLE_GUEST`):** Visitante sem autenticação. Acesso restrito à consulta e leitura da lista pública de planos de aula e atividades aprovadas.

### 2. Perfis de Atuação e Vínculo

* **Autor Original:** Professor responsável pela criação inicial de um plano de aula ou atividade no repositório.
* **Adaptador / Derivador:** Professor que utilizou um plano existente como base e gerou uma versão adaptada para a realidade da sua turma com auxílio da IA.
* **Avaliador / Relator:** Educador que aplicou o plano na prática e registrou um feedback/relato de experiência com fotos ou notas.

---

## 4. Modelo de Dados (Entidades Principais)

**[ENT01] - Usuário (User):**
Entidade para autenticação e gestão de acessos na plataforma.
* **Id:** (Inteiro) Identificador único do usuário.
* **Nome:** (Texto) Nome completo do usuário.
* **E-mail:** (Texto) E-mail institucional (obrigatoriamente terminado com `.br` para professores).
* **Senha:** (Hash) Credencial segura para autenticação local.
* **Perfil Global (Role):** (Enum) `ROLE_ADMIN`, `ROLE_SECRETARIA`, `ROLE_PROFESSOR_PENDING`, `ROLE_PROFESSOR`.
* **Status de Homologação:** (Enum) `PENDENTE`, `APROVADO`, `REJEITADO`, `BLOQUEADO`.
* **Data de Cadastro:** (Timestamp) Data de registro na plataforma.
* **Data de Homologação:** (Timestamp) Data em que o cadastro foi aprovado.
* **Homologado Por:** (Chave Estrangeira -> Usuário) Administrador ou Gestor de Secretaria que aprovou o cadastro.

**[ENT02] - Secretaria de Educação (EducationDepartment):**
Representa a entidade governamental de educação (municipal ou estadual).
* **Id:** (Inteiro) Identificador único da Secretaria.
* **Nome:** (Texto) Nome oficial da Secretaria (ex.: "Secretaria Municipal de Educação de Campinas").
* **Tipo:** (Enum) `MUNICIPAL`, `ESTADUAL`.
* **UF:** (Texto - 2 chars) Estado da federação.
* **Município:** (Texto) Nome da cidade (obrigatório para secretarias municipais).
* **Domínio Oficial:** (Texto) Padrão de domínio de e-mail da rede (ex.: `@educacao.sp.gov.br`).
* **Ativo:** (Booleano) Indicador de status ativo/inativo.

**[ENT03] - Vínculo Professor-Secretaria (TeacherAffiliation):**
Associação formal entre um professor e uma ou mais secretarias de educação.
* **Id:** (Inteiro) Identificador único do vínculo.
* **Id_Professor:** (Chave Estrangeira -> User) Identificador do professor.
* **Id_Secretaria:** (Chave Estrangeira -> EducationDepartment) Secretaria à qual o professor está vinculado.
* **Status:** (Enum) `PENDENTE`, `VINCULADO`, `DESVINCULADO`.
* **Data de Vinculação:** (Timestamp) Data em que a secretaria homologou a vinculação.

**[ENT04] - Habilidade e Competência BNCC (BnccSkill):**
Catálogo oficial das habilidades da BNCC tradicional e da BNCC da Computação.
* **Id:** (Inteiro) Identificador único.
* **Código:** (Texto) Código alfanumérico oficial da habilidade (ex.: `EM13CO01`, `EF05CI02`).
* **Descrição:** (Texto longo) Enunciado completo da habilidade/competência.
* **Eixo da Computação:** (Enum / Opcional) `PENSAMENTO_COMPUTACIONAL`, `MUNDO_DIGITAL`, `CULTURA_DIGITAL`, `NAO_APLICAVEL`.
* **Etapa de Ensino:** (Enum) `EDUCACAO_INFANTIL`, `FUNDAMENTAL_ANOS_INICIAIS`, `FUNDAMENTAL_ANOS_FINAIS`, `ENSINO_MEDIO`.
* **Ano / Faixa Etária:** (Texto) Ano escolar específico (ex.: "1º e 2º Anos", "6º Ano").
* **Componente Curricular:** (Texto) Disciplina vinculada (ex.: Computação, Matemática, Ciências, Língua Portuguesa).
* **Objeto Conhecimento:** (Texto) Objeto do conhecimento vinculado (ex.: Algoritmos, Hardware e software, Tipos de dados).

**[ENT05] - Plano de Aula / Atividade (LessonPlan):**
Entidade central do repositório contendo o conteúdo didático e metodológico.
* **Id:** (Inteiro) Identificador único do plano/atividade.
* **Título:** (Texto) Nome atrativo e descritivo da atividade.
* **Resumo:** (Texto longo) Sinopse dos objetivos e da proposta.
* **Tipo de Atividade:** (Enum) `PLANO_DE_AULA`, `ATIVIDADE_DESPLUGADA`, `PROJETO_MAKER`, `ATIVIDADE_DIGITAL`.
* **Etapa de Ensino:** (Enum) `EDUCACAO_INFANTIL`, `FUNDAMENTAL_INICIAIS`, `FUNDAMENTAL_FINAIS`, `ENSINO_MEDIO`.
* **Anos Indicados:** (Texto) Anos/séries adequados para a aplicação.
* **Duração Estimada:** (Texto) Tempo previsto (ex.: "2 aulas de 50 min").
* **Componentes Curriculares:** (Lista/Texto) Disciplinas envolvidas (ex.: Computação, Artes).
* **Materiais Necessários:** (Texto longo) Lista de insumos, recursos ou sucatas requeridas.
* **Metodologia / Passo a Passo:** (Texto longo em Markdown) Instruções detalhadas para execução com os alunos.
* **Critérios de Avaliação:** (Texto longo) Como o professor pode avaliar o aprendizado.
* **Imagem de Capa (Banner):** (URL/Arquivo) Imagem ilustrativa para o card na lista.
* **Status de Publicação:** (Enum) `RASCUNHO`, `PENDENTE_MODERACAO`, `PUBLICADO`, `ARQUIVADO`.
* **Id_Autor:** (Chave Estrangeira -> User) Professor criador.
* **Gerado Por IA:** (Booleano) Indicador se a estrutura do plano foi gerada via IA.
* **É Derivado:** (Booleano) Indicador se este plano foi adaptado a partir de outro.
* **Id_Plano_Origem:** (Chave Estrangeira -> LessonPlan / Opcional) Apontamento para o plano base em caso de derivação.
* **Visualizações:** (Inteiro) Contador de acessos.
* **Data de Criação / Atualização:** (Timestamp) Ciclo de vida temporal do registro.

**[ENT06] - Habilidade do Plano (LessonPlanSkill):**
Tabela associativa de relacionamento N:N entre Planos e Habilidades BNCC.
* **Id:** (Inteiro) Identificador único.
* **Id_Plano:** (Chave Estrangeira -> LessonPlan)
* **Id_Habilidade:** (Chave Estrangeira -> BnccSkill)

**[ENT07] - Contexto de Adaptação com IA (AiAdaptation):**
Registro do histórico de solicitações e parâmetros informados à IA para personalização de planos.
* **Id:** (Inteiro) Identificador único do ciclo de geração.
* **Id_Professor:** (Chave Estrangeira -> User) Educador solicitante.
* **Id_Plano_Base:** (Chave Estrangeira -> LessonPlan / Opcional) Plano base selecionado (se houver).
* **Id_Plano_Gerado:** (Chave Estrangeira -> LessonPlan / Opcional) Plano resultante salvo.
* **Materiais Disponíveis:** (Texto) Insumos reais descritos pelo professor.
* **Perfil da Turma:** (Texto) Nível de maturidade, tamanho da turma, comportamentos e preferências.
* **Habilidade Foco:** (Texto/Chave Estrangeira) Habilidade prioritária desejada.
* **Observações e Instruções Livres:** (Texto) Prompt ou diretrizes específicas fornecidas pelo professor.
* **Resposta Bruta da IA:** (Texto longo) Retorno estruturado recebido da LLM.
* **Tokens Consumidos:** (Inteiro) Métrica de custo/uso.
* **Data da Interação:** (Timestamp) Momento da geração.

**[ENT08] - Relato de Aplicação / Feedback Prático (ApplicationReport):**
Depoimento e evidências de professores que aplicaram o plano em sala.
* **Id:** (Inteiro) Identificador único.
* **Id_Plano:** (Chave Estrangeira -> LessonPlan) Plano que foi aplicado.
* **Id_Professor:** (Chave Estrangeira -> User) Professor que aplicou.
* **Relato de Experiência:** (Texto longo) Descrição de como foi a recepção dos estudantes, dificuldades e pontos fortes.
* **Avaliação Geral (Nota):** (Inteiro de 1 a 5) Classificação do plano.
* **Fotos / Evidências:** (JSON / Lista de URLs) Arquivos de fotos de trabalhos dos estudantes ou da sala.
* **Data da Aplicação:** (Data) Data em que a atividade ocorreu.
* **Data do Registro:** (Timestamp) Data de envio do relato.

**[ENT09] - Favorito / Coleção (Favorite):**
Planos salvos na biblioteca pessoal do professor.
* **Id:** (Inteiro) Identificador único.
* **Id_Usuario:** (Chave Estrangeira -> User) Professor logado.
* **Id_Plano:** (Chave Estrangeira -> LessonPlan) Plano favoritado.
* **Data do Registro:** (Timestamp) Data e hora da ação.

---

## 5. Requisitos Funcionais e Cenários de Uso

### **Módulo Externo (Público & Visitantes)**

#### **[CEN01] - Exploração da Lista de Planos e Atividades**
* **RF01:** O sistema deve exibir na lista pública todos os planos de aula e atividades com status "Publicado".
* **RF02:** O sistema deve apresentar os planos em formato de cards responsivos, contendo imagem de capa, título, autor, etapa de ensino, eixos da BNCC Computação e contador de favoritos/adaptações.
* **RF03:** O sistema deve permitir busca textual inteligente por título, descrição, palavras-chave, materiais ou código de habilidade BNCC.
* **RF04:** O sistema deve disponibilizar filtros rápidos combináveis por:
  - Etapa de Ensino (Educação Infantil, Anos Iniciais, Anos Finais, Ensino Médio);
  - Eixos da BNCC Computação (Pensamento Computacional, Mundo Digital, Cultura Digital);
  - Tipo de Atividade (Desplugada, Maker, Digital);
  - Disciplinas/Componentes Curriculares correlacionados;
  - Planos gerados/adaptados por IA vs. Planos manuais.
* **RF05:** O sistema deve disponibilizar a página completa de detalhes do plano, contendo introdução, habilidades BNCC completas, materiais, passo a passo, árvore genealógica de derivação (se for um plano adaptado de outro) e relatos de aplicação de outros professores.

---

### **Módulo do Professor (Criação, Adaptação e Repositório)**

#### **[CEN02] - Cadastro e Gestão Manual de Planos (Acesso Livre para Professores)**
* **RF06:** O sistema deve permitir que qualquer professor cadastrado (mesmo com status "Pendente") crie, edite e submeta planos de aula preenchendo os dados de forma manual.
* **RF07:** O sistema deve possibilitar a vinculação de uma ou mais habilidades da base da BNCC ao plano cadastrado através de um seletor assistido com busca por código e descrição.
* **RF08:** O sistema deve permitir o upload de imagem de capa (JPG/PNG) e anexos pedagógicos (PDFs de moldes, fichas de atividade ou matrizes de impressão).
* **RF09:** O sistema deve permitir salvar planos como "Rascunho" ou enviá-los para publicação na lista da comunidade.

#### **[CEN03] - Assistente DesplugAI Studio (Exclusivo para Professores Homologados)**
* **RF10:** O sistema deve disponibilizar o botão **"Adaptar com IA"** na página de detalhes de qualquer plano público para professores com status "Aprovado".
* **RF11:** O sistema deve disponibilizar o botão **"Criar Novo com IA"** para geração de planos inéditos a partir de diretrizes livres do professor homologado.
* **RF12:** O sistema deve fornecer um formulário intuitivo de contextualização de turma para a IA, coletando:
  - Materiais concretos disponíveis (ex.: "apenas papel sulfite, tampinhas de garrafa e barbante");
  - Perfil e maturidade da turma (ex.: "turma de 3º ano muito agitada, gostam de jogos de tabuleiro e movimento");
  - Tempo de aula disponível;
  - Habilidade BNCC foco a ser enfatizada;
  - Instruções adicionais livres do professor.
* **RF13:** O sistema deve processar a requisição de adaptação/geração através de um serviço inteligente de IA, retornando um plano estruturado (título, justificativa da adaptação, novo passo a passo contextualizado e rubrica de avaliação).
* **RF14:** O sistema deve permitir que o professor revise, edite livremente qualquer campo gerado pela IA e decida salvar a nova versão como um plano derivado vinculado ao plano original.
* **RF15:** O sistema deve bloquear o acionamento do DesplugAI Studio para professores não homologados, exibindo mensagem instrutiva sobre a necessidade de aprovação de seu e-mail institucional.

#### **[CEN04] - Biblioteca do Professor e Relatos de Aplicação**
* **RF16:** O sistema deve disponibilizar a área "Meu Repositório", onde o professor visualiza:
  - Meus Planos Publicados e Rascunhos;
  - Minhas Adaptações Geradas por IA;
  - Meus Planos Favoritos.
* **RF17:** O sistema deve permitir que o professor registre um "Relato de Aplicação" em planos que ele executou em sala, informando a data, nota (1 a 5), relato textual das reações dos estudantes e envio de fotos da aula.
* **RF18:** O sistema deve manter a atribuição ao autor original, exibindo em planos adaptados o selo: *"Adaptado com DesplugAI a partir do plano [Nome do Plano Original] de [Nome do Autor Original]"*.

---

### **Módulo da Secretaria de Educação (Governança e Relatórios)**

#### **[CEN05] - Homologação e Gestão de Professores da Rede**
* **RF19:** O sistema deve fornecer à Secretaria de Educação uma interface de busca de professores por nome ou e-mail institucional (com domínio `.br`).
* **RF20:** O sistema deve permitir que a Secretaria de Educação aprove e vincule um professor à sua rede de ensino, alterando seu status para "Aprovado" e liberando o uso do DesplugAI Studio.
* **RF21:** O sistema deve permitir que a Secretaria desvincule ou suspenda o vínculo de um professor de sua rede quando necessário.

#### **[CEN06] - Painel de Indicadores e Relatórios Pedagógicos da Rede**
* **RF22:** O sistema deve prover à Secretaria um Dashboard com métricas consolidadas dos professores vinculados à sua rede:
  - Total de professores cadastrados e homologados;
  - Quantidade de planos e atividades criadas (mensal, semestral e anual);
  - Quantidade de adaptações via IA realizadas pelos docentes;
  - Quantidade de relatos de aplicação prática registrados.
* **RF23:** O sistema deve exibir gráficos demonstrando as áreas de conhecimento, componentes curriculares e eixos da BNCC Computação mais trabalhados pelos professores da rede.
* **RF24:** O sistema deve permitir a exportação de relatórios consolidados em formato PDF e planilha estruturada (CSV/Excel) para prestação de contas pedagógicas e planejamento de formações continuadas.

---

### **Módulo de Administração Global (Governança Master)**

#### **[CEN07] - Moderação, Gestão de Secretarias e Catálogo BNCC**
* **RF25:** O sistema deve permitir ao Administrador cadastrar, editar e gerenciar Secretarias de Educação Municipais e Estaduais, associando seus gestores.
* **RF26:** O sistema deve permitir ao Administrador aprovar manualmente professores independentes (professores com e-mail `.br` cujo município/estado ainda não possui secretaria cadastrada na plataforma).
* **RF27:** O sistema deve disponibilizar painel de moderação para suspender, editar ou despublicar planos ou relatos que violem as diretrizes pedagógicas ou de proteção a dados.
* **RF28:** O sistema deve permitir a importação e atualização da base oficial de habilidades da BNCC e BNCC da Computação.

---

## 6. Requisitos Não Funcionais

* **[RNF01] - Tecnologia do Back-end:** API REST desenvolvida em Java (Spring Boot) com persistência em banco de dados relacional MySQL/PostgreSQL.
* **[RNF02] - Tecnologia do Front-end:** Interface web moderna desenvolvida em React/Next.js ou Vue.js, componentizada, acessível e otimizada para dispositivos móveis e desktops.
* **[RNF03] - Integração com Modelos de Linguagem (LLM):** Conexão robusta e assíncrona com API de IA Generativa (ex.: Google Gemini API / Vertex AI) utilizando engenharia de prompts com saída estruturada (JSON).
* **[RNF04] - Segurança e Autenticação:** Autenticação via JWT (JSON Web Tokens), controle de acessos baseado em papéis (RBAC - Role Based Access Control) e criptografia de senhas com algoritmo seguro (BCrypt).
* **[RNF05] - Armazenamento de Mídias e Arquivos:** Armazenamento seguro de imagens de capa, arquivos anexos (PDFs) e fotos de relatos práticos em bucket de objetos (ou diretório estruturado de arquivos com validação de formato e tamanho).
* **[RNF06] - Usabilidade e Acessibilidade:** Interface limpa e intuitiva segundo padrões WCAG (Web Content Accessibility Guidelines), com suporte a contraste e leitores de tela.
* **[RNF07] - Desempenho e Resiliência da IA:** A geração de planos com IA deve possuir tratamento de timeout, fallback visual com carregamento contextual (skeletons/spinners) e política de retry em caso de instabilidade na API de LLM.

---

## 7. Regras de Negócio e Validações

### **[RN01] - Validação Obrigatória de E-mail Institucional (.br)**
No cadastro de qualquer usuário com perfil de Professor, o sistema deve validar estritamente o formato do e-mail. É obrigatório que o domínio termine com a extensão geográfica nacional `.br` (ex.: `@escola.gov.br`, `@professor.sp.gov.br`, `@educacao.curitiba.pa.gov.br`, `@instituicao.edu.br`). Cadastros com e-mails comerciais comuns sem sufixo institucional serão rejeitados ou sinalizados para moderação.

### **[RN02] - Gate de Homologação para Acesso aos Recursos de IA**
O acesso às funcionalidades do **DesplugAI Studio** (gerar planos do zero com IA ou adaptar planos existentes com IA) é restrito a professores com `Status de Homologação = APROVADO`.
* **Professores Pendentes:** Podem navegar na lista, favoritar planos e criar planos de aula através do formulário manual.
* **Tentativa de Uso de IA:** Caso um professor pendente tente acionar a geração por IA, o sistema deve emitir o aviso: *"Funcionalidade exclusiva para professores homologados. Aguarde a validação do seu e-mail institucional pela Secretaria de Educação ou Administrador."*

### **[RN03] - Duplo Fluxo de Aprovação de Professores**
A homologação de um professor pode ocorrer por duas vias:
1. **Via Secretaria de Educação:** A Secretaria pesquisa o e-mail do professor na base e aciona o comando "Homologar e Vincular à Rede".
2. **Via Administrador Global:** O Administrador aprova diretamente professores avulsos cujas cidades ainda não possuem secretaria credenciada no sistema.

### **[RN04] - Isolamento de Dados e Visibilidade das Secretarias**
Uma Secretaria de Educação possui permissão de leitura e visualização de relatórios exclusivamente sobre os dados agregados e planos produzidos pelos professores que estejam formalmente vinculados à sua rede municipal/estadual. É vedado o acesso a dados individuais ou métricas detalhadas de outras secretarias.

### **[RN05] - Linhagem e Atribuição de Planos Adaptados (Fork Pedagógico)**
Ao gerar uma adaptação de um plano existente com o auxílio da IA:
1. O novo plano deve obrigatoriamente manter a referência ao `Id_Plano_Origem`.
2. O autor original deve ser creditado publicamente no card e no cabeçalho do plano adaptado.
3. O plano original não é sobrescrito, preservando o trabalho do autor inicial.

### **[RN06] - Alinhamento Mandatório à BNCC da Computação**
Todo plano cadastrado manualmente ou gerado/adaptado pela IA deve conter a indicação de pelo menos uma competência/habilidade da BNCC tradicional ou BNCC da Computação associada, garantindo o rigor e a relevância pedagógica do repositório.

### **[RN07] - Engenharia de Prompt e Proteção de Dados de Estudantes (LGPD)**
O formulário de contexto de turma do DesplugAI Studio não deve coletar nomes próprios ou dados sensíveis identificáveis de estudantes individuais. A interface deve exibir aviso explícito: *"Não insira nomes de alunos ou dados pessoais identificáveis no campo de contexto de turma. Descreva apenas características coletivas da turma e necessidades pedagógicas gerais."* 

### **[RN08] - Regras de Upload de Imagens e Anexos**
* **Imagens de Capa e Relatos:** Formatos aceitos `.jpg`, `.jpeg`, `.png`, `.webp`, com tamanho máximo de 5MB por arquivo.
* **Anexos Pedagógicos (Moldes, Fichas):** Formato aceito exclusivamente `.pdf`, com tamanho máximo de 10MB por arquivo.
* **Sanitização de Arquivos:** Todos os arquivos enviados devem ser renomeados com hash/UUID para evitar conflitos de nomenclatura e ataques de travessia de diretório.
* **Deve conter um aviso explícito para que não seja realizado upload de fotos de estudantes em nenhuma hipótese. Os uploads devem ser de materias e atividades que não permita identificar nenhum estudante.**

### **[RN09] - Moderação e Diretrizes de Conteúdo**
Planos publicados na comunidade que recebam denúncias de conteúdo inadequado, plágio descaracterizado ou desvio das finalidades educacionais podem ser revertidos a qualquer momento para o status `PENDENTE_MODERACAO` por Administradores ou Gestores de Secretaria autorizados, saindo imediatamente da lista pública.


# Controle de versões 

| Versão | Data | Descrição | Responsável | Revisado por |
| --- | --- | --- | --- | --- |
| 0.1 | 01/09/2026 | Versão inicial do documento |  [Lairson Alencar](https://github.com/lerao/) | - |

