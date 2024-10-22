# MulVAL - Multi-host, Multi-stage Vulnerability Analysis Tool

MulVAL is a tool for generating attack graphs to help visualize vulnerabilities across multiple hosts in a network. It uses the XSB logic engine and GraphViz to create detailed attack graphs based on input from tools like OVAL and Nessus.

## Prerequisites

To run MulVAL, you need:

1. **XSB Logic Engine**: Install from [XSB](http://xsb.sourceforge.net/).
2. **GraphViz**: Verify installation by running `dot`. If not installed, get it from [GraphViz](http://www.graphviz.org/).

Ensure both `xsb` and `dot` are available in your system's PATH.

## Setup

1. Set the environmental variable `MULVALROOT` to the root folder of this package.
2. Add `$MULVALROOT/bin` and `$MULVALROOT/utils` to your PATH.
3. Run `make` to compile everything.

## Running MulVAL

You can either run MulVAL's attack-graph generator directly if you have an input file, or you can use the adapters to create input files before generating the attack graph.

### Direct Execution

```bash
graph_gen.sh INPUT_FILE [OPTIONS]
```

A sample input file is available in `testcases/3host/input.P` for a 3-host example. Run it using:

```bash
graph_gen.sh input.P -v -p
```

> **Note:** The `-p` option is for visualization and should not be used in production as it can significantly slow down the process.

By default, MulVAL outputs the attack graph in both **textual** (`AttackGraph.txt`) and **XML** (`AttackGraph.xml`) formats. If the `-v` option is used, it will also generate a **PDF** attack graph using GraphViz (`AttackGraph.pdf`).

### Options

- **Graph Generation Options**:
  - `-l`: Output the attack graph in `.CSV` format.
  - `-v`: Output the attack graph in `.CSV` and `.PDF` formats.
  - `-p`: Perform deep trimming on the attack graph (for visualization, **not for production**).

- **Reasoning Options**:
  - `-r | --rulefile RULE_FILE`: Use a custom interaction ruleset.
  - `-a | --additional ADDITIONAL_RULE_FILE`: Use additional rule files with the default ruleset.
  - `-g | --goal ATTACK_GOAL`: Specify a single attack goal.
  - `--cvss`: Use CVSS information from the input file.
  - `-ma`: Group CVSS information and perform risk assessment.

- **Rendering Options**:
  - `--arclabel`: Display arc labels.
  - `--reverse`: Reverse the order of arcs.
  - `--nometric`: Do not show metric information.
  - `--simple`: Do not show vertex fact labels (useful for large attack graphs).
  - `--nopdf`: Skip PDF generation and output only the DOT file.

### Running MulVAL with Rendering Options

After running the `graph_gen.sh` script, you can invoke `render.sh` to apply rendering options:

```bash
render.sh [RENDERING_OPTIONS]
```

## Preparing MulVAL Input Files Using Adapters

### Steps

1. **Setup MySQL Database for NVD Data**:
   - Set up an empty MySQL database to store NVD data.
   - Update `config.txt` with database connection info:
     ```
     jdbc:mysql://www.abc.edu:3306/nvd
     user_name
     password
     ```
   - Run `nvd_sync.sh` to sync the database.

2. **Translate OVAL/Nessus Reports to Datalog Format**:
   - For OVAL reports:
     ```bash
     oval_translate.sh XML_REPORT_FROM_IN_OVAL
     ```
   - For Nessus reports:
     ```bash
     nessus_translate.sh XML_NESSUS_REPORT [FIREWALL_RULES]
     ```
     > Optional: Add firewall rules in the second parameter.

3. **Create `hacl` Tuples**:
   - Define host-to-host connectivity using `hacl(Host1, Host2, Protocol, Port)` in the input file.

4. **Generate Attack Graph**:
   - Follow instructions in the **Running MulVAL** section to generate the attack graph.

## Advanced Usage

1. **Custom Rule Set**:
   - You can create custom rules using interaction rule files. Use the `-r` option to load a custom rule file:
     ```bash
     graph_gen.sh input.P -r my_interaction_rules.P
     ```

2. **Risk Metrics Calculation**:
   - To calculate risk metrics based on CVSS and MulVAL attack graphs, use:
     ```bash
     probAssess.sh
     ```

   - Or use the combined script:
     ```bash
     riskAssess.sh INPUT [OPTIONS]
     ```

## References

1. Xinming Ou, Wayne F. Boyer, and Miles A. McQueen. [A Scalable Approach to Attack Graph Generation](https://dl.acm.org/doi/10.1145/1180405.1180471). In ACM Conference on Computer and Communications Security (CCS), 2006.
2. Xinming Ou, Sudhakar Govindavajhala, and Andrew W. Appel. [MulVAL: A Logic-based Network Security Analyzer](https://www.usenix.org/conference/14th-usenix-security-symposium/mulval-logic-based-network-security-analyzer). In USENIX Security Symposium, 2005.
3. Su Zhang, Xinming Ou, and John Homer. [Effective Network Vulnerability Assessment Through Model Abstraction](https://link.springer.com/chapter/10.1007/978-3-642-22629-7_14). In DIMVA, 2011.
4. Lingyu Wang et al. [An Attack Graph-Based Probabilistic Security Metric](https://link.springer.com/chapter/10.1007/978-3-642-18519-8_15). In IFIP WG 11.3, 2008.
