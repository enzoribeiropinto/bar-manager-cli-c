<div align="center">

# 🍽️ Bar Manager

**A student bar management system in C: prepaid balances, purchase history and file persistence, built on hand-crafted dynamic data structures.**

![Language](https://img.shields.io/badge/language-C-00599C?style=for-the-badge&logo=c&logoColor=white)
![Dependencies](https://img.shields.io/badge/dependencies-none-success?style=for-the-badge)
![Persistence](https://img.shields.io/badge/storage-plain%20text-informational?style=for-the-badge)
![Status](https://img.shields.io/badge/status-functional-brightgreen?style=for-the-badge)
![License](https://img.shields.io/badge/license-MIT-lightgrey?style=for-the-badge)

[Features](#-features) • [How It Works](#-how-it-works) • [Getting Started](#-getting-started) • [Data Format](#-data-file-format) • [Roadmap](#%EF%B8%8F-roadmap)

</div>

---

## 📖 About

Bar Manager is a command-line application for running a university bar. Staff can register students, top up their prepaid balance, record purchases, review spending history and list everyone alphabetically. Everything is saved to disk on exit and restored on the next launch.

The whole system is written in **standard C with no external libraries**: the data structures, the file parser and the memory management are all implemented from scratch.

## ✨ Features

| | Feature | Details |
|---|---|---|
| 👤 | **Register students** | Name, birth date, course, year, student number, initial credit |
| 🗑️ | **Remove students** | Search by **name** or **student number**; all their data is freed |
| 🔤 | **Alphabetical listing** | Sorted view with number, course and balance |
| 🛒 | **Record purchases** | Validates price and available balance, then logs description and date |
| 💳 | **Top up balance** | Rejects zero and negative amounts |
| 🧾 | **Spending history** | Per-student itemised list with running total |
| 💾 | **Automatic persistence** | Loads `dados.txt` on start, saves on exit |

## 🧠 How It Works

### Data model

```
Queue ──► Node ──► Node ──► Node ──► NULL
 front      │
 rear       └─► aluno
                 ├── nome, curso, ano, numero, dinheiro
                 ├── nascimento (data)
                 └── compras ──► CompraNode ──► CompraNode ──► NULL
                                  (preço, descrição, data)
```

| Structure | Purpose |
|---|---|
| `Queue` | FIFO list of students with `front` and `rear` pointers, giving **O(1)** insertion |
| `Node` | One student in the queue |
| `aluno` | Student record that owns its own purchase history |
| `CompraNode` | Linked list of purchases per student (one-to-many) |

### Design decisions

- **Three-layer structure.** Types in `fonte.h`, the public API in `function.h`, implementation in `functions.c`, and the user interface in `main.c`.
- **One deletion routine, two criteria.** `eliminar_aluno` takes a `POR_NOME` / `POR_NUMERO` flag instead of duplicating the pointer-relinking logic.
- **Leak-free by construction.** Helpers such as `libertar_compras` and `libertar` release every purchase list and node, on deletion and on program exit.
- **Sorting without side effects.** Listing builds a temporary array of pointers and sorts it with `qsort` and a custom comparator, leaving the queue order untouched.
- **Robust file loading.** The parser tells record types apart by counting field separators, reports malformed lines with the offending text, and survives a missing file by starting empty.
- **Guarded transactions.** Purchases are rejected for invalid students, non-positive prices or insufficient funds before anything is allocated or changed.
- **Safe input.** Bounded `scanf` formats (`%99[^\n]`) prevent buffer overflows and allow names with spaces; every `malloc` is checked.

## 🚀 Getting Started

### Prerequisites

- A C compiler (GCC or Clang)

### Build & run

```bash
# Clone the repository
git clone https://github.com/<your-username>/bar-manager.git
cd bar-manager

# Compile
gcc -Wall main.c functions.c -o bar

# Run
./bar
```

### Menu

```text
---Menu---
1. Adicionar aluno
2. Eliminar aluno
3. Listar alunos alfabeticamente
4. Realizar compra
5. Carregar dinheiro
6. Listar despesas de um aluno
0. Sair
```

<details>
<summary><b>Example: alphabetical listing</b></summary>

```text
--- Lista de Alunos (Ordem Alfabética) ---
Nome: Ana Costa                  | Num: 10234  | Curso: Informática     | Saldo: 12.50
Nome: Bruno Silva                | Num: 10587  | Curso: Engenharia      | Saldo: 5.00
Nome: Carla Mendes               | Num: 10911  | Curso: Gestão          | Saldo: 20.00
------------------------------------------
```

</details>

<details>
<summary><b>Example: spending history</b></summary>

```text
--- Despesas do aluno Ana Costa ---
1. Sandes mista         | 2.50 EUR | 14/05/2026
2. Café                 | 0.80 EUR | 15/05/2026
Total gasto: 3.30 EUR
-----------------------------
```

</details>

## 💾 Data File Format

Data lives in a human-readable `dados.txt` next to the executable. Each student line is followed by that student's purchases:

```text
nome|dd/mm/aaaa|curso|ano|numero|saldo      ← student  (5 separators)
preço|descrição|dd/mm/aaaa                    ← purchase (2 separators)
```

```text
Ana Costa|12/03/2003|Informática|2|10234|12.50
2.50|Sandes mista|14/05/2026
0.80|Café|15/05/2026
Bruno Silva|01/09/2002|Engenharia|3|10587|5.00
```

## 📁 Project Structure

```
.
├── fonte.h        # Data model: data, compra, CompraNode, aluno, Node, Queue
├── function.h     # Function declarations and search-criteria constants
├── functions.c    # Core logic: queue ops, search, sorting, transactions, file I/O
├── main.c         # Interactive menu
├── dados.txt      # Persisted data (created on first save)
└── README.md
```

## 🛣️ Roadmap

- [x] Student queue with O(1) insertion
- [x] Delete by name / student number
- [x] Alphabetical listing with `qsort`
- [x] Purchases and top-ups with balance validation
- [x] Per-student spending history
- [x] File persistence (load/save)
- [ ] Input validation for dates and numeric fields
- [ ] Prevent duplicate student numbers
- [ ] Makefile and automated tests
- [ ] Spending reports (by period, by student)

## 🧰 Tech Stack

| Area | Details |
|---|---|
| **Language** | C |
| **Concepts** | Pointers, structs, `typedef`, dynamic memory (`malloc`/`free`), header files, multi-file compilation |
| **Data structures** | Linked lists, queues, nested structures |
| **Algorithms** | Linear search, `qsort` with custom comparators, string parsing |
| **I/O** | File handling (`fopen`/`fgets`/`fprintf`), formatted parsing with `sscanf` |
| **Practices** | Modular design, defensive programming, error handling, leak prevention, DRY refactoring |
| **Tooling** | GCC, command line, Git |

## 🤝 Contributing

Suggestions and pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

1. Fork the project
2. Create your branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

Distributed under the MIT License. See `LICENSE` for details.

## 👤 Author

**Daniela Silva and Enzo Pinto**
[GitHub](https://github.com/<enzoribeiropinto>) • [LinkedIn](https://linkedin.com/in/enzo-ribeiro-pinto>)


<div align="center">

⭐ If you found this project interesting, consider giving it a star!

</div>
