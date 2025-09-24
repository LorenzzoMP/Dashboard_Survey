# Dashboard de Inspeção de Sistemas de Incêndio

Este projeto é uma solução de front-end desenvolvida como parte de um desafio técnico para a vaga de Programador Júnior. A aplicação consiste numa dashboard interativa que permite a um líder de equipa de inspeções visualizar e gerir as atividades da sua equipa de forma eficiente.

A dashboard cumpre três objetivos principais, conforme a User Story:
1.  **Visão Geral:** Apresenta um resumo do estado atual de todas as inspeções (Total, Concluídas, Pendentes).
2.  **Identificação de Urgências:** Mostra um gráfico que agrupa os alertas urgentes por cliente, permitindo uma rápida identificação das áreas problemáticas.
3.  **Planeamento Futuro:** Exibe um calendário com as inspeções agendadas para os próximos 30 dias.

## Tecnologias Utilizadas

* **Vue.js 3:** Utilizado como o framework principal para a construção da interface reativa.
* **Tailwind CSS:** Responsável por toda a estilização da aplicação, através de classes utilitárias.
* **Vite:** Ferramenta de build e servidor de desenvolvimento local.
* **Chart.js:** Biblioteca utilizada para a renderização dos gráficos.

## Como Executar o Projeto

Siga os passos abaixo para executar a aplicação localmente.

### Pré-requisitos

* Node.js (versão 20.x ou superior, conforme especificado no `package.json`).
* NPM (geralmente instalado com o Node.js).

### Passos para a Instalação

1.  **Clone o repositório para a sua máquina local:**
    ```sh
    git clone https://github.com/LorenzzoMP/Dashboard_Survey.git
    ```

2.  **Navegue até a pasta do projeto:**
    ```sh
    cd fire-inspection-dashboard
    ```

3.  **Instale todas as dependências do projeto:**
    ```sh
    npm install
    ```

### Rodando o Servidor de Desenvolvimento

1.  **Para iniciar a aplicação em modo de desenvolvimento, execute:**
    ```sh
    npm run dev
    ```

2.  Após a compilação, o terminal irá mostrar o endereço local para aceder à aplicação. Geralmente é:
    `http://localhost:5173`

3.  Abra este endereço no seu navegador para visualizar a dashboard.