# Concurrent Thetris
### SME0140 - Sistemas Operacionais
---
* **Docente:** Kalinka Castelo Branco
* **Alunos:** Pedro Henrique de Sousa Prestes (15507819) e Luysa de Souza Gonçalves (15474077)
---

## 1. Visão Geral

O **Concurrent Tetris** é uma implementação multijogador competitiva do clássico jogo Tetris, desenvolvida em C++ utilizando a biblioteca `ncurses` para renderização em terminal.

O diferencial deste projeto é sua **arquitetura concorrente**, que utiliza múltiplas threads para gerenciar input, lógica de jogo e renderização de forma assíncrona. Isso garante que o processamento pesado de um jogador ou o tempo de espera de entrada (input) não interfira na fluidez da animação ou na jogabilidade do oponente.

### Mecânicas de Jogo
* **Modo Competitivo:** Dois tabuleiros lado a lado (Jogador 1 vs Jogador 2).
* **Sistema de Ataque (Garbage):** Ao completar 2 ou mais linhas simultaneamente, você envia "lixo" (linhas cinzas com buracos) para a base do adversário, empurrando o jogo dele para cima.
* **Fim de Jogo:** O jogo termina quando as peças de qualquer um dos jogadores atingirem o topo da tela.

---

## 2. Implementação Técnica

O projeto evita um "Game Loop" sequencial único. Em vez disso, ele orquestra **4 threads independentes** para desacoplar operações bloqueantes da renderização contínua.

### 2.1 Arquitetura de Threads
A função `Game::run()` instancia as seguintes linhas de execução:

1.  **Thread de Input (`t_input`):** Atua como **Produtor**. Fica em loop bloqueante aguardando teclas e insere comandos em uma fila segura.
2.  **Thread de Renderização (`t_render`):** Atua como **Observador**. Lê o estado das matrizes (protegidas por semáforos) e desenha na tela a uma taxa fixa].
3.  **Threads de Lógica (`t_player1` & `t_player2`):** Atuam como **Consumidores**. Cada jogador possui sua thread que calcula física, gravidade e consome comandos da fila de input.

### 2.2 Sincronização e Semáforos (C++20)
O projeto utiliza primitivas de sincronização do C++20 (`<semaphore>`) para garantir a integridade dos dados e evitar *Race Conditions*.

| Tipo de Semáforo | Uso no Projeto |
| :--- | :--- |
| **`std::binary_semaphore`** | **Exclusão Mútua (Mutex):** Protege Regiões Críticas, como o acesso aos tabuleiros (durante a renderização/atualização) e filas de input. |
| **`std::counting_semaphore`** | **Sinalização:** Gerencia a mecânica de ataque. O atacante incrementa o semáforo (sinaliza lixo enviado) e a vítima consome (recebe linhas de lixo) sem bloquear se não houver ataque. |

---

## 3. Instalação e Execução

### 3.1 Pré-requisitos
O jogo foi projetado para ambientes **Unix-like** (Linux nativo ou WSL no Windows).

* **Compilador C++:** Suporte a **C++20** (ex: g++ versão 10 ou superior).
* **CMake:** Versão 3.16 ou superior.
* **Bibliotecas:** `ncurses` e `pthread`.

**Instalação de dependências (Ubuntu/Debian/WSL):**
```bash
sudo apt update
sudo apt install build-essential cmake libncurses5-dev libncursesw5-dev g++
```

### 3.2 Compilação
Navegue até a pasta raiz do projeto e escolha um método:
```bash
mkdir build
cd build
cmake ..
make
```

### 3.3 Executando o Jogo
Após a compilação, execute o binário:
```bash
./tetris
```

### 4 Controles
| Ação | Jogador 1 (Esq.) | Jogador 2 (Dir.) |
| :--- | :---: | :---: |
| Mover Esquerda | A  | J  |
| Mover Direita | D | L |
| Girar Peça | W | I |
| Acelerar Descida | S | K |
| Sair do Jogo | Q | Q |
