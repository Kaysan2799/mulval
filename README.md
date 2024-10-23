
# MulVAL - Multi-host, Multi-stage Vulnerability Analysis Tool

MulVAL is a scalable, logic-based network security analyzer that can generate attack graphs from an enterprise network setup. This tool leverages the XSB logic engine to reason about the vulnerabilities in a network and uses GraphViz to visualize the attack graph.

## Prerequisites

1. **XSB Logic Engine**: [Download XSB](http://xsb.sourceforge.net/)
2. **GraphViz**: Check if GraphViz is installed by typing `dot` in your terminal. If not installed, [download GraphViz](http://www.graphviz.org/).

Make sure both `xsb` and `dot` are included in your system `PATH`.

## Setup

1. Clone this repository and set the environment variable `MULVALROOT` to the package’s root folder.
2. Include `$MULVALROOT/bin` and `$MULVALROOT/utils` in your system `PATH`.
3. Run `make` to compile the necessary components.

## Running MulVAL

You can run MulVAL directly using the attack-graph generator if you already have an input file. If not, use the provided adapters to create the input files.

### Example:

Run the simple input file in `testcases/3host/input.P`:

```bash
graph_gen.sh input.P -v -p
```

This will generate an attack graph based on the provided input file. Note that the `-p` option **should not be used in production**, as it may slow down the process.

By default, MulVAL outputs the following files:

- **Textual Format**: `AttackGraph.txt`
- **XML Format**: `AttackGraph.xml`
- **PDF Format**: `AttackGraph.pdf` (only if `-v` option is used)

To view the attack graph visually, open the `AttackGraph.pdf` file.

## Options

### Graph Generation Options:

- `-l`: Output attack-graph in `.CSV` format
- `-v`: Output attack-graph in `.CSV` and `.PDF` format
- `-p`: Perform deep trimming on the attack graph (not recommended for production)

### Reasoning Options:

- `-r | --rulefile RULE_FILE`: Use the specified rule set
- `-a | --additional ADDITIONAL_RULE_FILE`: Use additional rule files
- `-g | --goal ATTACK_GOAL`: Specify a single attack goal
- `--cvss`: Use CVSS data in the input file
- `-ma`: Use CVSS data with grouping (requires a grouped input file)

### Rendering Options:

- `--arclabel`: Output labels for the arcs
- `--reverse`: Reverse the order of the arcs
- `--nometric`: Do not show metric information
- `--simple`: Simplify vertex fact labels (for large graphs)
- `--nopdf`: Do not generate the PDF (only the `.dot` file)

## Preparing MulVAL Input Files Using Adapters

MulVAL provides several adapter programs to help create input files from enterprise network data.

### Steps:

1. **Set up MySQL for NVD**:
   Create a new MySQL database and add connection info to `config.txt`.

   Example:
   ```
   jdbc:mysql://www.abc.edu:3306/nvd
   user_name
   password
   ```

   Run the `nvd_sync.sh` script to populate the NVD database.

2. **Translate OVAL/Nessus Reports**:
   - **For OVAL**:
     ```bash
     oval_translate.sh XML_REPORT_FROM_IN_OVAL
     ```
     Output files: `oval.P`, `summ_oval.P`, `grps_oval.P`
   - **For NESSUS**:
     ```bash
     nessus_translate.sh XML_NESSUS_REPORT [FIREWALL_RULES]
     ```
     Output files: `nessus.P`, `summ_nessus.P`, `grps_nessus.P`

3. **Create hacl Tuples**:
   Define connection information in the format `hacl(Host1, Host2, Protocol, Port)` in the input file. Combine translated input files into a single file.

4. **Generate Attack Graph**:
   Use the `graph_gen.sh` script to generate the attack graph.

## Advanced Usage

### Custom Rule Sets

You can create your own interaction rules by writing a new rule file (e.g., `my_interaction_rules.P`). Test it by running:

```bash
graph_gen.sh INPUT_FILE -r my_interaction_rules.P
```

### Risk Assessment Using CVSS and MulVAL

To calculate probabilistic risk metrics based on the attack graph and CVSS scores, run the following command in the directory containing the attack graph:

```bash
probAssess.sh
```

Alternatively, use `riskAssess.sh` to combine the steps of generating an attack graph and running the risk assessment algorithm:

```bash
riskAssess.sh INPUT [OPTIONS]
```

## References

1. Xinming Ou, Wayne F. Boyer, and Miles A. McQueen, *A scalable approach to attack graph generation*, 13th ACM CCS, 2006.
2. Xinming Ou, Sudhakar Govindavajhala, and Andrew W. Appel, *MulVAL: A logic-based network security analyzer*, 14th USENIX Security Symposium, 2005.
3. Su Zhang, Xinming Ou, and John Homer, *Effective network vulnerability assessment through model abstraction*, DIMVA, 2011.
4. Lingyu Wang et al., *An attack graph-based probabilistic security metric*, DBSEC'08, 2008.
