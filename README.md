# Concurrent Tetris

Implementação multijogador competitiva do Tetris, em C++ com `ncurses`, construída sobre uma arquitetura concorrente com threads e semáforos.

<p align="center">
  <img alt="Disciplina" src="https://img.shields.io/badge/SSC0140-Sistemas%20Operacionais-blue?style=for-the-badge">
  <img alt="Instituição" src="https://img.shields.io/badge/USP-Universidade%20de%20S%C3%A3o%20Paulo-orange?style=for-the-badge">
</p>

> **Disciplina:** SSC0140 - Sistemas Operacionais <br>
> **Docente:** Kalinka Castelo Branco <br>
> **Instituição:** USP - Universidade de São Paulo

<p align="center">
  <img src="https://img.shields.io/badge/C++20-00599C?style=for-the-badge&logo=cplusplus&logoColor=white" alt="C++20" />
  <img src="https://img.shields.io/badge/CMake-064F8C?style=for-the-badge&logo=cmake&logoColor=white" alt="CMake" />
  <img src="https://img.shields.io/badge/ncurses-4D4D4D?style=for-the-badge" alt="ncurses" />
</p>

## Visão Geral

O diferencial do projeto é a **arquitetura concorrente**: múltiplas threads gerenciam input, lógica de jogo e renderização de forma assíncrona. O processamento pesado de um jogador, ou a espera por input, não interfere na fluidez da animação nem na jogabilidade do oponente.

### Mecânicas de Jogo

- **Modo competitivo:** dois tabuleiros lado a lado (Jogador 1 vs Jogador 2).
- **Sistema de ataque (garbage):** ao completar 2 ou mais linhas simultaneamente, o jogador envia "lixo" (linhas cinzas com buracos) para a base do adversário, empurrando o jogo dele para cima.
- **Fim de jogo:** termina quando as peças de qualquer jogador atingem o topo da tela.

## Requisitos

O jogo foi projetado para ambientes **Unix-like** (Linux nativo ou WSL no Windows).

- **Compilador C++** com suporte a **C++20** (por exemplo, `g++` 10 ou superior)
- **CMake** 3.16 ou superior
- Bibliotecas `ncurses` e `pthread`

```bash
sudo apt update
sudo apt install build-essential cmake libncurses5-dev libncursesw5-dev g++
```

## Como Executar

```bash
mkdir build
cd build
cmake ..
make
./tetris
```

### Controles

| Ação | Jogador 1 (Esq.) | Jogador 2 (Dir.) |
| :--- | :---: | :---: |
| Mover Esquerda | A | J |
| Mover Direita | D | L |
| Girar Peça | W | I |
| Acelerar Descida | S | K |
| Sair do Jogo | Q | Q |

## Estrutura do Projeto

- `main.cpp`: ponto de entrada.
- `Game.cpp` / `Game.h`: laço do jogo e criação das threads.
- `Board.cpp` / `Board.h`: tabuleiro e peças.
- `Constants.h`: constantes do jogo.
- `CMakeLists.txt`: build.

## Implementação Técnica

O projeto evita um "game loop" sequencial único e orquestra **4 threads independentes**, que desacoplam operações bloqueantes da renderização contínua. A função `Game::run()` instancia:

1. **Thread de input (`t_input`):** produtor. Fica em loop bloqueante aguardando teclas e insere comandos em uma fila segura.
2. **Thread de renderização (`t_render`):** observador. Lê o estado das matrizes (protegidas por semáforos) e desenha na tela a uma taxa fixa.
3. **Threads de lógica (`t_player1` e `t_player2`):** consumidoras. Cada jogador tem sua thread, que calcula física e gravidade e consome comandos da fila de input.

### Sincronização e Semáforos (C++20)

O projeto usa as primitivas de `<semaphore>` do C++20 para garantir a integridade dos dados e evitar *race conditions*.

| Tipo de semáforo | Uso no projeto |
| :--- | :--- |
| `std::binary_semaphore` | **Exclusão mútua:** protege regiões críticas, como o acesso aos tabuleiros (renderização e atualização) e as filas de input. |
| `std::counting_semaphore` | **Sinalização:** gerencia a mecânica de ataque. O atacante incrementa o semáforo (sinaliza lixo enviado) e a vítima consome (recebe as linhas) sem bloquear se não houver ataque. |

## Autores

- Pedro Henrique de Sousa Prestes
- Luysa de Souza Gonçalves
