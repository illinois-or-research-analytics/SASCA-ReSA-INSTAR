# SASCA-ReSA-INSTAR

The _Scalable Agent-based Simulator for Citation Analysis SASCA(-ReSA)_ is a scalable agent-based modeling simulator that can begin with a small seed network and simulate an exponential network growth to reach sizes of 100 million nodes and more. Currently, SASCA is implemented in modern C++ and can be run across hundreds of cores.

**SASCA-ReSA-INSTAR** is derived from [SASCA-ReSA](https://github.com/illinois-or-research-analytics/SASCA-ReSA) 6.2.8 (commit `5fc654b`), whose full history this repository carries. It makes the exponents of the agent scoring functions configurable instead of fixed at 3 (see [Score exponents](#score-exponents-and-notation)). With the new keys omitted, it behaves exactly like SASCA-ReSA 6.2.8, so existing configuration files run unchanged. (An *instar* is the developmental stage between molts.)

## Recent changelogs
6.4.0 (SASCA-ReSA-INSTAR): Optional `generator_eligibility` flag (`and`, the default and 6.2.8 behavior, or `or`) sets whether a non-random generator must pass both the in-degree and fitness thresholds or either one. The size of the eligible set is logged each year. If the eligible set is empty, agents now fall back to a random generator (previously undefined behavior). Seeds are now ranked by their own fitness (1 by default, as used in citation scoring) in the eligibility test; before, they were ranked as `fitness_value_max` during the first `recency_threshold` years. This changes results only when `non_random_generator_probability` > 0. Optional `generator_pool` (`all`, the default; `agents_eligible`; `agents`) restricts generators to agents: only the non-random generators, or all generators. In the first simulated year, before any agents exist, all nodes remain eligible.
 See [Generator selection](#generator-selection) for how this behaves in the first years of a run.

6.3.0 (SASCA-ReSA-INSTAR): Score exponents and offset are configurable through the optional `score_exponent`, `pa_exponent`, `fitness_exponent`, `num_authors_exponent`, `author_reputation_exponent`, and `score_offset` flags (previously a constant `gamma = 3` and offset 1). Defaults reproduce 6.2.8. Removes an unused exponent lookup table.

6.2.8: Adds out-degree safeguarding based on the constant in the header file of abm and the input out-degree bag file.

6.2.7: Fixes a bug with initial author reputation when only one node exists in the final year of the seed set and a brand new author gets assigned to it

6.2.5: Adds support for cartel authors

6.2.4: Fixes file flushing on WriteGraph()

6.2.2: Fixes a bug with number of authors and planted nodes feature where number of authors was not having an effect

6.2.1: functionality did not change but the output nodelist now has initial author reputation and final author reputation

6.2.0: introduces author layer with h-index as author reputation and updated planted nodes structure

6.1.0: introduces author layer and log-based score calculation


## Dependencies
- C++ >= 20
- OpenMP >= 4.0
- cmake >= 3.23
- Eigen3 (can be locally installed via [setup.sh](setup.sh))
- PCG (can be locally installed via [setup.sh](setup.sh))
- inih (can be locally installed via [setup.sh](setup.sh))
- argparse (can be locally installed via [setup.sh](setup.sh))

## One time setup
Run [setup.sh](setup.sh) to locally install [Eigen](https://eigen.tuxfamily.org/index.php?title=Main_Page), [PCG](https://www.pcg-random.org/), [inih](https://github.com/jtilly/inih.git), and [argparse](https://github.com/p-ranav/argparse.git). Alternatively, just ensure that these libraries are discoverable by cmake and adjust the build accordingly.

## How to build
SASCA is a standard cmake project. [easy_build_and_compile.sh](easy_build_and_compile.sh) is provided for user convenience.

### Running in a cluster environment
Below is the recommended way of building and running a standard cmake project on a cluster environment such as the Illinois Campus Cluster. This will ensure that the code is compiled with the right optimizations for the specific compute node it is assigned.
```console
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release <SASCA git root>
make
cd ..
./build/bin/abm --config <configuration file>
rm -r ./build
```

### Release mode node on the Illinois Campus Cluster
Building the project under the Release mode requires no modules to be loaded.

### Debug mode node on the Illinois Campus Cluster
Building the project under the Debug mode requires the ASAN libray. This library is not loaded properly on the campus cluster by default. For those using the Illinois Campus Cluster, load any `gcc` module version greater than 11 e.g., `module load gcc/12.4.0`.

## How to run
The general command is given below.
```console
abm --config <Configuration file>
```

An example configuration file, shown below, is also provided in the docs folder [Sample configuration file](docs/SASCA-ReSA-INSTAR.sample.config.ini) (a working 30-year baseline run on the multifield seed, with every flag commented)
Brief comments for each flag is listed inside the example configuration file. Detailed explanations for each of the flags are at the end of this section. All flags are required to be present in the configuration file, except those marked optional (the score exponent flags), which may be omitted entirely. Unused flags can simply be left empty but the flag itself must still be present.


```
[Environment]
nodelist=<FILE> ; csv with (node_id, publication_year) on each line
edgelist=<FILE> ; csv with (source,target) on each line
recency_table=<FILE> ; csv derived from a real-world network
out_degree_bag=<FILE> ; csv derived from a real-world network
growth_rate=<DOUBLE> ; floating point value e.g., 0.03 for 3%
num_cycles=<INT> ; integer value e.g., 30 for a 30-year simulation
recency_bins=<STRING> ; string with comma separated values for each bin
planted_agents=<(optional) FILE> ; csv with planted agent with a planted agent per line. See below for the list of supported columns.
start_from_checkpoint=<BOOL> ; boolean value e.g., true or false for whether to start from a checkpoint or not

[Agent]
alpha=<FLOAT> ; floating point value specifying the alpha for neighborhood
use_alpha=<BOOL> ; boolean value e.g., true or false for whether to use alpha or not
same_year_citations=<DOUBLE> ; floating point value e.g., 0.12 for 12%
fully_random_citations=<DOUBLE> ; floating point value e.g., 0.05 for 5%
preferential_weight=<DOUBLE> ; floating point value e.g., 0.33
fitness_weight=<DOUBLE> ; floating point value e.g., 0.33
num_authors_weight=<DOUBLE> ; floating point value e.g., 0.33
author_reputation_weight=<DOUBLE> ; floating point value e.g., 0.33
fitness_value_min=<INT> ; minimum fitness value (inclusive)
fitness_value_max=<INT> ; maximum fitness value (inclusive)
fitness_lag_duration_min=<INT> ; minimum lag duration in years (inclusive)
fitness_lag_duration_max=<INT> ; maximum lag duration in years (inclusive)
fitness_peak_duration_min=<INT> ; minimum peak duration in years (inclusive)
fitness_peak_duration_max=<INT> ; maximum peak duration in years (inclusive)
neighborhood_sample=<INT> ; maximum number of nodes to sample from each neighborhood
in_degree_threshold=<INT> ; used for non-random generator selection. Selects the top <INT> percentile nodes by in-degree value
fitness_threshold=<INT> ; used for non-random generator selection. Selects the top <INT> percentile nodes by fitness value
recency_threshold=<INT> ; used for non-random generator selection. Selects only from the past <INT> years
non_random_generator_probability=<DOUBLE> ; used for non-random generator selection. 0.9 for 90% of the nodes picking non-random generators
author_max_lifetime=<INT> ; used for the maximum amount of years that an author is allowed to publish
num_authors_bag=<FILE> ; used for sampling the number of authors per paper
cartel_outdegree_proportion=<DOUBLE> ; the proportion of out-degree that should go towards within the cartel group. See below for important cartel information.
null_cartel=<BOOL> ; boolean value e.g., true or false for whether to have cartel nodes cite randomly within the cartel or not. See below for important cartel information.
clonal_cartel_agent_file<(optional) FILE> ; csv with clonal agent attributes. See below for important cartel information.
score_exponent=<(optional) DOUBLE> ; exponent for all four agent scores, default 3. See Score exponents below.
pa_exponent=<(optional) DOUBLE> ; exponent for the preferential attachment (in-degree) score, default score_exponent
fitness_exponent=<(optional) DOUBLE> ; exponent for the fitness score, default score_exponent
num_authors_exponent=<(optional) DOUBLE> ; exponent for the number-of-authors score, default score_exponent
author_reputation_exponent=<(optional) DOUBLE> ; exponent for the author reputation score, default score_exponent
score_offset=<(optional) DOUBLE> ; constant added to every score, default 1
generator_eligibility=<(optional) and|or> ; non-random generators must pass the in-degree AND fitness thresholds (default) or either one
generator_pool=<(optional) all|agents_eligible|agents> ; which nodes may be generators: all (default), agents only for non-random generators, or agents only for all generators
generator_pool_min_agent_share=<(optional) DOUBLE> ; with an agents-only pool, filter seeds out of the eligible set once agents are at least this share of it, default 0.1

[General]
output_file=<FILE> ; output csv edgelist
auxiliary_information_file=<FILE> ; output auxiliary information file
log_file=<FILE> ; output log file
num_processors=<INT> ; integer valued maximum parallelism allowed
log_level=<INT> ; 0, 1, and 2 for silent, info, and verbose
```

For example, a simulation with no planted agents can be done using a configuration file structured as below.
```
...
recency_bins=1,5,10
planted_agents=
start_from_checkpoint=false
...
```

In some cases, such as when `use_alpha` is false, the value for `alpha` must be empty.

```
...
[Agent]
alpha=
use_alpha=false
...
```
### Null model
In order to do a "null model" run, in which agents cite fully randomly within the neighborhood, set the `recency_bins` string to be "1" and nothing else. This leads to a binning where all nodes with age at least 1 get binned into the same single bin. Moreover, since this single bin is also the last bin, everything in this bin will be cited in a fully random manner.

### Single-bin model
In order to do a "single-bin model" run, in which agents cite based on preferental attachment and fitness within the neighborhood without regards to recency, set the `recency_bins` string to be "1,`some large number`". This leads to a binning where all nodes with age at least 1 and less than `some large number` get binned into the same bin and then any node from `some large number` to infinity gets binned in another bin. This ensures that the [1, `some large number`) bin is not the last bin in the simulation. Therefore, it will cite essentially all nodes according to preferential attachment and fitness only.

### Individual flags
#### Environment flags
- `nodelist`: csv file with (node\_id, publication\_year) on each line. Both headers and their values are required. This is the starting seed network nodelist. When starting from a checkpoint, so when `start_from_checkpoint` is true, then `nodelist` supplied should be the auxiliary information file from the previous run with no modifications.
- `edgelist`: csv file with (source,target) on each line. This is the starting seed network edgelist. When starting from a checkpoint, so when `start_from_checkpoint` is true, then `edgelist` supplied should be the output file from the previous run with no modifications. Note that the header line for this CSV does not actually matter as long as the first column represents the source node and the second columns repreresns the target node. For example, the header line could be source,target or citing,cited, or u,v or any valid csv row.
- `recency_table` csv file derived from a real-world network. This file is usually created by taking a real-world citation network and getting the count Y which equals the number of edges that represent a citation across x years. This is encoded as a row in the csv such as x,y for each x. Note that for any simulation, a row for x needs to exist for all x from 1 to last year in the simulation - oldest publication year in the seed node set. Note that the header line for this CSV is required but the string values of the header line is not enforced. The only requirement is that starting from the second line of the csv that there are two comma separated integer values for each line.
- `out_degree_bag`: csv file derived from a real-world network. This file is usually created by taking real-world citation network and getting the out-degree of each node in the network. This is encoded as a row in the csv such as node\_id,out\_degree for each node in the network. Note that the node\_id column is unused but is useful for keeping track of which real-world publication was used. The headers in this file are not enforced except that two columns are required and that the second column must be integers. The largest out-degree specified in this file must be at most the value specified in the `max\_out\_degree` constant defined the header file for abm. Increasing this constant will increase the memory usage.
- `growth_rate`: floating point value, e.g., 0.03 for 3%, which serves as the exponent for the exponential growth formula used in determining how many new agents should spawn per cycle of simulation.
- `num_cycles`: integer value e.g., 30 for a 30-year simulation
- `recency_bins`: This string is a user supplied comma separated string such that each comma separated value in the string is the start of the bin boundary. For example, a string "1,2,5,10,20" represents a binning for recency such that the first bin contains all publications that are less than 2 year old, meaning 1 <= current year - publication year < 2. It follows that the second bin now are publications that are at least 2 years old but at most 4 years old (2 <= current year - publication year < 5). The last bin is implied to go on until infinity so in this example, the last bin contains all publications that are at least 20 years old (20 <= current\_year - publication year < infinity).
- `planted_agents`: an optional csv file with planted agents. The only required column is the "year" column. The year column specifies the relative year, from the start of the simulation, when the node should be planted. 1 is the earliest relative year an agent can be planted. Any column header here must match the exact name in the output nodelist. Only supported columns are "pa_weight", "fit_weight", "num_authors_weight", "author_reputation_weight", "out_degree", "alpha", "fit_lag_duration", "fit_peak_value", "fit_peak_duration", "num_authors", and "author_id."
- `start_from_checkpoint` boolean value e.g., true or false for whether to start from a checkpoint or not. Check notes about `nodelist` and `edgelist` if this flag is set to true.

#### Agent flags
- `alpha` floating point value specifying the alpha for neighborhood. This value for alpha determines the proportion of citations that are made to the 1-hop nodes of the generator node relative to the total number of citations that the agent will make to the generator node's neighborhood. It can be left to be -1 for a random model or a constant value such as 0.5.
- `use_alpha` boolean value e.g., true or false for whether to use alpha or not. When alpha i
- `same_year_citations`: floating point value e.g., 0.12 for 12%. This value determines the proportion of new agents in a given year that cite another agent from the same year.
- `fully_random_citations`: floating point value e.g., 0.05 for 5%. This value determines the proportion of assigned out-degree that goes towards citing any node from the graph in a fully random manner.
- `preferential_weight`: floating point value e.g., 0.33. This value can be left -1 for agents with random weights or a constant value for static agents.
- `fitness_weight`: floating point value e.g., 0.33. This value can be left -1 for agents with random weights or a constant value for static agents.
- `num_authors_weight`: floating point value e.g., 0.33. This value can be left -1 for agents with random weights or a constant value for static agents.
- `author_reputation_weight`: floating point value e.g., 0.33. This value can be left -1 for agents with random weights or a constant value for static agents.
- `fitness_value_min`: minimum possible integer fitness value for each agent.
- `fitness_value_max`: maximum possible integer fitness value for each agent.
- `fitness_lag_duration_min`: minimum lag duration in years (inclusive). The fitness decay model in SASCA-ReSA works as follows. For the first `x` number of years, an agent is dormant and has the lowest possible fitness value. Afterwards, the agent immediately gains its full fitness value and retains it for `y` number of years. Finally, the fitness power for the agent decays according to a decaying power-law curve. `x` and `y` here are drawn from uniform distributions built by the lag duration min/max values and peak duration min/max values, respectively.
- `fitness_lag_duration_max`: maximum lag duration in years (inclusive). See `fitness_lag_duration_min`.
- `fitness_peak_duration_min`: minimum peak duration in years (inclusive).See `fitness_lag_duration_min`.
- `fitness_peak_duration_max`: maximum peak duration in years (inclusive). See `fitness_lag_duration_min`.
- `neighborhood_sample`: maximum number of nodes to sample from each neighborhood. The neighborhoods of the genartor will be sampled such that the final list of nodes is at most this number.
- `in_degree_threshold`: used for non-random generator selection. Selects the top percentile nodes by in-degree value, meaning that the top nth percentile node is found by in-degree, afterwhich all nodes with in-degree value at least that value is selected.
- `fitness_threshold`: used for non-random generator selection. Selects the top percentile nodes by fitness value, meaning that the top nth percentile node is found by fitness, afterwhich all nodes with fitness value at least that value is selected.
- `recency_threshold`: used for non-random generator selection. Selects only from the past n years e.g., if `recency_threshold=5` and current year is 1988, then 5 years of publications will be considered so the nodes with publication year at least 1983 and at most 1987.
- `non_random_generator_probability`: Probability of each node to select a generator node in a non-random manner according to the different thresholds. Set to 0 for fully random and 1 for all non-random.
- `generator_eligibility`: optional, `and` (default) or `or`. How the in-degree and fitness criteria combine when building the set of eligible non-random generators. Each year, the papers within the `recency_threshold` window are ranked by in-degree and by fitness; a paper passes a criterion if its value is at least that of the paper at the threshold percentile, so ties at the cutoff are all included (with most fitness values equal to 1, the fitness criterion can admit noticeably more than `fitness_threshold` percent). With `and` a paper must pass both, with `or` either. Seed papers are ranked by their own fitness (1 by default); early in a run, when the window holds mostly fitness-1 seeds, the tie rule can let nearly every paper pass the fitness criterion. Each agent that draws a non-random generator picks one uniformly from this set; the set's size is written to the log each year.
- `generator_pool`: optional, `all` (default), `agents_eligible`, or `agents`. Restricts which nodes can be generators. With `agents_eligible`, seeds are removed from the eligible set of non-random generators; random generators are still drawn from all nodes. With `agents`, both non-random and random generators are drawn from agents only. In the first simulated year there are no agents yet, so all nodes are used. Pool sizes are written to the log each year.
- `generator_pool_min_agent_share`: optional, between 0 and 1, default 0.1. With an agents-only pool, seeds are removed from the eligible set only once agents are at least this share of it; before that the unfiltered eligible set is used. See [Generator selection](#generator-selection).
- `author_max_lifetime`: used for the maximum amount of years that an author is allowed to publish. When this value is `k`, the difference between the publication year of the last paper and the publication year of the first paper of any given author cannot exceed `k`.
- `num_authors_bag` : used for sampling the number of authors per paper. This is a CSV wherelthe header line is ignored and each subsequent line is structed as (index,number of authors). The index column is not used, and the number of authors column is used to sample a number for the number of authors for each paper.
- `cartel_outdegree_proportion` : the proportion of out-degree that should go towards within the cartel group. Even if there are no cartel authors, make sure to set this to some value between 0 and 1. It will not be used if there are no cartel authors. **See below for important cartel information.**
- `null_cartel` : boolean value e.g., true or false for whether to have cartel nodes cite randomly within the cartel or not. Set this to either true or falses even if there are no cartel authors. It will not be used if there are no cartel authors. True means "cartel-r" and False means "cartel-p". **See below for important cartel information.**
- `clonal_cartel_agent_file` : csv with clonal agent attributes. The allowed columns are num\_authors,pa\_weight,fit\_weight,num\_authors\_weight,author\_reputation\_weight,out\_degree,alpha,fitness\_lag\_duration,fitness\_peak\_value, and fitness\_peak\_duration. The order of the columns do not matter, and any subset of these columns can be used and others omitted in the csv file. All cartel agents (cartel-r, cartel-p, or control) inherit these attributes. ( **See below for important cartel information.**

- `score_exponent`, `pa_exponent`, `fitness_exponent`, `num_authors_exponent`, `author_reputation_exponent`, `score_offset`: optional. See [Score exponents and notation](#score-exponents-and-notation).

#### General flags
- `output_file`: output csv edgelist file.
- `auxiliary_information_file`: output auxiliary information file a.k.a. the nodelist. This file contains the information for each node such as assigned out-degree, assigned alpha values, assigned fitness values, final in-degree, and so on.
- `log_file`: output log file. This log file contains information about the run itself such as the runtime. The configuration file flags will be parsed and the parsed results will be written to this file before the simulation begins. Additionally, an extra file is created in the same directory as the log file containing the wall-time of individual stages inside the simulator.
- `num_processors`: integer valued maximum parallelism allowed. Used for setting the maximum number of workers.
- `log_level`: enum flag where 0, 1, and 2 correspond to silent, info, and verbose, respectively.

## Cartel Information
The codebase itself never adds authors to a cartel. When a brand new simulation begins without starting from a checkpointed output, the nodelist is read in and author ids are assigned to each node in the nodelist. In this use case, no cartel publications will be generated for the entirety of the simulation.

When a simulation begins from a checkpointed output, the author id and cartel id columns are read in from the nodelist, and this is the only process that defines the set of authors that will be in a cartel. Therefore, to initiate a cartel simulation, one must start from a brand new simulation for at least 0 years, modify the output nodelist manually, and then do a second simulation from the modified output as a checkpointed run.

When modifying the output nodelist manually, there are two key details that must be followed.
1. Pick the set of authors first from the set of author ids already present in the nodelist.
2. For every publication authored by those authors, set the cartel id column to be either 1 or 0. 1 indicates that it is a true cartel whose behavior is further defined by the null cartel flag. When null cartel is true, the behavior of the cartel authors will follow "cartel-r". When null cartel is false, the behavior of the cartel authors will follow "cartel-p." See individual flag descriptions for more detail. 0, however, indicates that it is a control cartel, meaning that these authors will be guaranteed to publish a publication each year but will behave as background agents for all other aspects (e.g., generator nodes are not always other cartel nodes).

Remember that
- Cartel nodes have other cartel nodes as their generator nodes
- Cartel nodes will try to cite a publication from each of the cartel members but will prioritize citing from those cartel author publications that are within the 2-hop neighborhood first.
- Whether a cartel node is in a control group or not is determined through the input nodelist `cartel_id` column, i.e., control if 0.
- Whether a non-control cartel node is cartel-r or cartel-p is determined through the `null_cartel` flag.


## Fitness Information
For constant fitness, set minimum/maximum fitness lag duration to be 0 and minimum/maximum fitness peak duration to be 1000.


## Score exponents and notation
When an agent chooses whom to cite among a set of candidate nodes, each candidate gets four raw scores computed from its in-degree, fitness, number of authors, and author reputation. For a raw value x, the score is

```
score(x) = max(x^β, 1) + C
```

normalized to sum to 1 over the network, and the four normalized scores are mixed using the agent's weights (`pa_weight`, `fit_weight`, `num_authors_weight`, `author_reputation_weight`). In SASCA-ReSA 6.2.8 and earlier, β was the constant `gamma = 3` and C was 1 for all four scores. Here they are set by:

| flag | symbol | applies to | default |
|---|---|---|---|
| `pa_exponent` | β_P | in-degree (preferential attachment) | `score_exponent` |
| `fitness_exponent` | β_F | fitness | `score_exponent` |
| `num_authors_exponent` | β_A | number of authors | `score_exponent` |
| `author_reputation_exponent` | β_R | author reputation | `score_exponent` |
| `score_exponent` | — | all four, unless overridden | 3 |
| `score_offset` | C | all four | 1 |

Exponents must lie in [0, 10] (larger values overflow double precision for large in-degrees or fitness values) and C must be non-negative. The values used are written to the log file at startup. Note the floor: with C = 1, x = 0 and x = 1 both score 2.

**Notation.** Three different quantities are easily confused, so we name them separately:
- **α** (`alpha`) — the locality parameter: the fraction of neighborhood citations made to distance-1 nodes of the generator. Unchanged.
- **β** — the exponent of the attachment rule above, an *input*. β = 1 is linear preferential attachment as in Price (1976) and Barabási & Albert (1999); β > 1 is superlinear.
- **γ** — the tail exponent of the resulting in-degree distribution, N(k) ∼ k^(−γ), an *output*. Redner (1998) measured γ ≈ 3 for real citation networks (he called it α).

SASCA-ReSA and earlier models use β = 3 for all four scores, a superlinear design choice for an artificial generative model; it remains the default so earlier results are reproducible. INSTAR makes β configurable so its effect can be studied: β = 1 corresponds to the linear rule of Price (1976) and Barabási & Albert (1999), and `pa_exponent=1` sets it for the in-degree score. Note that β (an input) is distinct from the in-degree tail exponent γ (an output), e.g. Redner's (1998) γ ≈ 3. Because candidates are restricted to sampled neighborhoods and recency bins, cited without replacement, and scored by a mixture of four terms, the effective attachment kernel of the grown network can differ substantially from β and should be measured rather than assumed. A first comparison of β_P = 1 against β = 3 is in [docs/beta_comparison.md](docs/beta_comparison.md).


## Generator selection
Each new agent gets one *generator* paper; its citation candidates are drawn from the generator's 1- and 2-hop neighborhood (both directions). How the generator is chosen:

1. Draw u ~ U(0,1). If u < `non_random_generator_probability` and the **eligible set** (below) is non-empty, pick a generator uniformly from the eligible set.
2. Otherwise pick uniformly from the **generator pool**: all nodes (`generator_pool=all` or `agents_eligible`), or all agents (`generator_pool=agents`).
3. Cartel authors are handled separately and always use cartel papers.

**Eligible set**, recomputed every year:
- Start from the papers published within the last `recency_threshold` years (the *window*).
- *In-degree criterion*: papers whose current in-degree is at least that of the paper at the top `in_degree_threshold` percent of the window.
- *Fitness criterion*: the same for current fitness and `fitness_threshold`.
- Combine with `generator_eligibility`: `and` (both criteria) or `or` (either).
- If `generator_pool` is `agents_eligible` or `agents`, remove seeds from the eligible set, but only once agents make up at least `generator_pool_min_agent_share` (default 10%) of it. Before that (in the first year there are no agents; in early years the eligible papers are mostly seeds), the unfiltered eligible set is used, seeds included.

Cutoffs are ranks within the whole window (seeds and agents together), and ties at the cutoff are all included. Seeds are ranked by their own fitness, which is 1 unless the nodelist says otherwise.

### Early years: what actually happens
The rules above interact with the seed network in the first years of a run, so "non-random" selection does not mean the same thing throughout. The log reports the window size, how many papers pass each criterion, the eligible set size and (with `generator_pool`) the agent pool each year, so the following can be checked for any run. Example: the multifield seed (26,686 seeds, 8,915 of them published in the 10 years before the first simulated year), `recency_threshold=10`, `in_degree_threshold=5`, `fitness_threshold=20`, `non_random_generator_probability=0.5`:

| year | window | pass in-degree | pass fitness | eligible (`and`) | eligible papers that are agents |
|---|---|---|---|---|---|
| 1 | 8,915 | 469 | all | 469 | no agents yet: all nodes used, 469 seeds eligible |
| 2 | 11,563 | 614 | all | 614 | **0** (pool: 3,582 year-1 agents) |
| 3 | 14,738 | 760 | all | 760 | **15** (pool: 7,644) |
| 4 | 18,384 | 924 | all | 924 | **69** |
| 5 | 22,668 | 1,135 | all | 1,135 | 184 |
| 6 | 27,646 | 1,407 | 5,920 (21%) | 184 | 184 |
| 10 | 56,771 | 2,968 | 14,083 (25%) | 1,071 | 1,071 |
| 30 | 736,785 | 38,428 | 184,304 (25%) | 16,115 | 16,115 |

(From one `generator_eligibility=and` run with `generator_pool=agents` and `fully_random_citations=0.01`, made before the share rule below, when the filtered set was used as is; other runs differ slightly.)

What this means:
- **Fitness passes everything at first.** While the window is mostly fitness-1 seeds, the fitness cutoff is 1 and every paper ties at it. Through year 5 here, `and` means "top 5% by in-degree" and `or` means "any recent paper", i.e. close to random selection. The fitness criterion becomes selective once papers with fitness above 1 make up more than `fitness_threshold` percent of the window; with mostly fitness-1 papers, about 21–25% then pass, not exactly 20%.
- **With `generator_pool=agents` or `agents_eligible`, agents reach the eligible set slowly.** The in-degree cutoff is set by the seeds in the window, and a new paper has few citations, so in the run above the eligible papers that are agents numbered 0 in year 2, 15 in year 3, 69 in year 4 and 184 in year 5. Filtering out seeds in those years would leave the non-random half of new agents with no generator (year 2) or all sharing a few (in year 3, about 2,300 agents and 15 generators), concentrating early citation. The filter is therefore applied only once agents are at least `generator_pool_min_agent_share` (default 10%) of the eligible set: in an 8-year test with this rule the agent share of the eligible set was 0%, 0%, 3% and 11% in years 1-4, so the filter started in year 4 (earlier than in the table above, which was made without the fallback: with seeds also acting as generators in the early years, agents accumulate citations a little faster). A share rather than a year count keeps this meaningful at other growth rates, where seeds stay a large part of the eligible set for longer; the share is tunable, and a high value keeps the fallback on for longer. Until then the unfiltered eligible set, mostly seeds, is used, as in year 1. Each year's log says which applied. Random draws with `generator_pool=agents` use agents from year 2 on.
- **Possible variant (not implemented):** with an agents-only pool, the in-degree and fitness cutoffs could be ranked among the agents in the window rather than among all papers, so the eligible set would be non-empty from year 2.

When comparing conditions, keep in mind that the generator rules change behavior over the first ~5–10 years of a run. Measures computed on early cohorts mix the two regimes.
