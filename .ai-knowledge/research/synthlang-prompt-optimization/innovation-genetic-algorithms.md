# Innovation: Genetic Algorithms for Prompt Optimization

**Category**: Innovation
**Source**: SynthLang, GAAPO research, EvoPrompt
**Key Innovation**: Treat prompts as evolving genomes that improve through natural selection

---

## Overview

Genetic algorithms (GAs) apply principles of biological evolution to optimize prompts: **mutation**, **crossover**, **selection**, and **reproduction**. Instead of manual refinement, prompts evolve automatically toward optimal performance.

The innovation: **Prompts as genomes**. Each "generation" of prompts is slightly different. The fittest survive and reproduce. Over many generations, prompts become highly optimized for their specific task.

SynthLang reports achieving **70% token reduction** and **233% faster processing** using this approach.

---

## Core Genetic Algorithm Concepts

### 1. Prompt as Genome

**Traditional view**: Prompt is static text
**GA view**: Prompt is a genome with mutable genes

```markdown
## Prompt Genome Structure

Gene 1: Instruction format (prose, list, table, XML, math)
Gene 2: Detail level (high, medium, low)
Gene 3: Example count (0, 1, 2, 3, 5)
Gene 4: Tone (direct, friendly, technical, formal)
Gene 5: Structure (linear, hierarchical, modular)
Gene 6: Specific phrasing (word choices)
...

Each gene can mutate independently
Combinations of genes create unique "organisms" (prompt variants)
```

**Example genome encoding**:
```typescript
interface PromptGenome {
  format: 'prose' | 'list' | 'table' | 'xml' | 'math';
  detailLevel: 'high' | 'medium' | 'low';
  exampleCount: 0 | 1 | 2 | 3 | 5;
  tone: 'direct' | 'friendly' | 'technical' | 'formal';
  structure: 'linear' | 'hierarchical' | 'modular';
  phrases: {
    opening: string;
    rules: string;
    examples: string;
    closing: string;
  };
}

// Example genome
const genome1: PromptGenome = {
  format: 'list',
  detailLevel: 'medium',
  exampleCount: 2,
  tone: 'technical',
  structure: 'hierarchical',
  phrases: {
    opening: "Implement authentication:",
    rules: "Follow Firebase best practices",
    examples: "See examples below:",
    closing: "Ensure security."
  }
};
```

### 2. Fitness Function

**Key concept**: Define what "good" means mathematically.

**Multi-objective fitness**:
```typescript
interface FitnessMetrics {
  taskSuccess: number;      // 0-1: Did it complete the task?
  quality: number;          // 0-1: How well?
  tokenEfficiency: number;  // 0-1: How few tokens?
  clarity: number;          // 0-1: Clear instructions?
  adherence: number;        // 0-1: Does Claude follow it?
}

function fitnessFunction(
  prompt: string,
  testCases: TestCase[]
): number {
  const metrics: FitnessMetrics = evaluatePrompt(prompt, testCases);

  // Weighted combination
  const fitness = (
    0.40 * metrics.taskSuccess +      // Most important
    0.25 * metrics.quality +
    0.15 * metrics.tokenEfficiency +
    0.10 * metrics.clarity +
    0.10 * metrics.adherence
  );

  return fitness;  // 0-1, higher is better
}
```

**SynthLang's fitness components**:
1. **Clarity** - Is the prompt unambiguous?
2. **Specificity** - Does it cover edge cases?
3. **Task completion** - Does it achieve the goal?
4. **Token efficiency** - How compact is it?

### 3. Mutation

**Concept**: Random small changes to explore variations.

**Mutation types**:

```typescript
type MutationType =
  | 'format-change'        // prose → list
  | 'detail-adjustment'    // high → medium
  | 'phrase-rephrase'      // reword instructions
  | 'example-add'          // add an example
  | 'example-remove'       // remove an example
  | 'structure-change'     // linear → hierarchical
  | 'tone-shift'           // technical → direct
  | 'compression'          // remove redundancy
  | 'expansion';           // add detail

function mutate(
  genome: PromptGenome,
  mutationRate: number = 0.1
): PromptGenome {
  const mutated = { ...genome };

  // Each gene has mutationRate chance to mutate
  if (Math.random() < mutationRate) {
    mutated.format = randomChoice(['prose', 'list', 'table', 'xml', 'math']);
  }

  if (Math.random() < mutationRate) {
    mutated.detailLevel = randomChoice(['high', 'medium', 'low']);
  }

  if (Math.random() < mutationRate) {
    mutated.exampleCount = randomChoice([0, 1, 2, 3, 5]);
  }

  // Phrase mutations are more complex
  if (Math.random() < mutationRate) {
    mutated.phrases.opening = rephrase(genome.phrases.opening);
  }

  return mutated;
}
```

**Example mutation sequence**:
```markdown
Generation 0 (original):
"When implementing authentication, follow Firebase best practices.
Include comprehensive error handling and session management."
Format: prose, Detail: high, Examples: 0
Tokens: 25

↓ Mutation: format-change (prose → list)

Generation 1:
Authentication implementation:
- Follow Firebase best practices
- Include comprehensive error handling
- Manage sessions properly
Format: list, Detail: high, Examples: 0
Tokens: 18

↓ Mutation: detail-adjustment (high → medium) + compression

Generation 2:
Authentication:
- Firebase best practices
- Error handling
- Session management
Format: list, Detail: medium, Examples: 0
Tokens: 12

↓ Mutation: example-add

Generation 3:
Authentication:
- Firebase best practices
- Error handling
- Session management

Example: auth.signIn() → check session
Format: list, Detail: medium, Examples: 1
Tokens: 16
```

### 4. Crossover (Recombination)

**Concept**: Combine successful prompts to create offspring.

**Single-point crossover**:
```typescript
function crossover(
  parent1: PromptGenome,
  parent2: PromptGenome
): [PromptGenome, PromptGenome] {
  // Choose crossover point
  const crossoverPoint = randomInt(0, geneCount);

  // Swap genes after crossover point
  const child1 = {
    format: parent1.format,
    detailLevel: parent1.detailLevel,
    // ↓ Crossover point
    exampleCount: parent2.exampleCount,
    tone: parent2.tone,
    structure: parent2.structure,
    phrases: parent2.phrases
  };

  const child2 = {
    format: parent2.format,
    detailLevel: parent2.detailLevel,
    // ↓ Crossover point
    exampleCount: parent1.exampleCount,
    tone: parent1.tone,
    structure: parent1.structure,
    phrases: parent1.phrases
  };

  return [child1, child2];
}
```

**Uniform crossover** (each gene independently):
```typescript
function uniformCrossover(
  parent1: PromptGenome,
  parent2: PromptGenome
): PromptGenome {
  return {
    format: Math.random() < 0.5 ? parent1.format : parent2.format,
    detailLevel: Math.random() < 0.5 ? parent1.detailLevel : parent2.detailLevel,
    exampleCount: Math.random() < 0.5 ? parent1.exampleCount : parent2.exampleCount,
    tone: Math.random() < 0.5 ? parent1.tone : parent2.tone,
    structure: Math.random() < 0.5 ? parent1.structure : parent2.structure,
    phrases: {
      opening: Math.random() < 0.5 ? parent1.phrases.opening : parent2.phrases.opening,
      rules: Math.random() < 0.5 ? parent1.phrases.rules : parent2.phrases.rules,
      examples: Math.random() < 0.5 ? parent1.phrases.examples : parent2.phrases.examples,
      closing: Math.random() < 0.5 ? parent1.phrases.closing : parent2.phrases.closing
    }
  };
}
```

**Example crossover**:
```markdown
Parent 1 (good at clarity):
Format: table
Detail: high
Examples: 2
Quality: High clarity, verbose

Parent 2 (good at efficiency):
Format: xml
Detail: low
Examples: 0
Quality: Low tokens, less clear

↓ Crossover: Take best of both

Child:
Format: table (from Parent 1 - clear)
Detail: low (from Parent 2 - efficient)
Examples: 1 (compromise)
Result: Clear AND efficient
```

### 5. Selection

**Concept**: Choose which prompts survive to next generation.

**Tournament selection** (SynthLang's approach):
```typescript
function tournamentSelection(
  population: PromptGenome[],
  fitnessScores: number[],
  tournamentSize: number = 3
): PromptGenome {
  // Randomly select tournamentSize individuals
  const tournament: number[] = [];
  for (let i = 0; i < tournamentSize; i++) {
    tournament.push(randomInt(0, population.length));
  }

  // Return the fittest from tournament
  const winner = tournament.reduce((best, current) =>
    fitnessScores[current] > fitnessScores[best] ? current : best
  );

  return population[winner];
}
```

**Roulette wheel selection** (fitness-proportional):
```typescript
function rouletteSelection(
  population: PromptGenome[],
  fitnessScores: number[]
): PromptGenome {
  const totalFitness = fitnessScores.reduce((a, b) => a + b, 0);
  const pick = Math.random() * totalFitness;

  let cumulative = 0;
  for (let i = 0; i < population.length; i++) {
    cumulative += fitnessScores[i];
    if (cumulative >= pick) {
      return population[i];
    }
  }

  return population[population.length - 1];
}
```

**Elitism** (always keep the best):
```typescript
function elitistSelection(
  population: PromptGenome[],
  fitnessScores: number[],
  eliteCount: number = 2
): PromptGenome[] {
  // Sort by fitness
  const sorted = population
    .map((genome, i) => ({ genome, fitness: fitnessScores[i] }))
    .sort((a, b) => b.fitness - a.fitness);

  // Return top eliteCount
  return sorted.slice(0, eliteCount).map(x => x.genome);
}
```

---

## Genetic Algorithm Workflow

### Complete GA Loop

```typescript
interface GAConfig {
  populationSize: number;      // e.g., 20
  generations: number;         // e.g., 50
  mutationRate: number;        // e.g., 0.1
  crossoverRate: number;       // e.g., 0.7
  eliteCount: number;          // e.g., 2
  tournamentSize: number;      // e.g., 3
}

async function geneticAlgorithmOptimization(
  initialPrompt: string,
  testCases: TestCase[],
  config: GAConfig
): Promise<PromptGenome> {

  // 1. Initialize population
  let population: PromptGenome[] = [];
  const seed = parsePromptToGenome(initialPrompt);

  // Create diverse initial population
  for (let i = 0; i < config.populationSize; i++) {
    population.push(mutate(seed, 0.5));  // High initial mutation for diversity
  }

  // 2. Evolution loop
  for (let gen = 0; gen < config.generations; gen++) {
    console.log(`Generation ${gen}...`);

    // 3. Evaluate fitness
    const fitnessScores: number[] = [];
    for (const genome of population) {
      const prompt = genomeToPrompt(genome);
      const fitness = await fitnessFunction(prompt, testCases);
      fitnessScores.push(fitness);
    }

    // 4. Log best of generation
    const bestIdx = fitnessScores.indexOf(Math.max(...fitnessScores));
    console.log(`  Best fitness: ${fitnessScores[bestIdx].toFixed(3)}`);
    console.log(`  Best genome: ${JSON.stringify(population[bestIdx], null, 2)}`);

    // 5. Check convergence
    const avgFitness = fitnessScores.reduce((a, b) => a + b) / fitnessScores.length;
    if (fitnessScores[bestIdx] > 0.95) {
      console.log(`Converged at generation ${gen}!`);
      break;
    }

    // 6. Selection for next generation
    const nextGeneration: PromptGenome[] = [];

    // 6a. Elitism - keep best performers
    const elite = elitistSelection(population, fitnessScores, config.eliteCount);
    nextGeneration.push(...elite);

    // 6b. Create offspring
    while (nextGeneration.length < config.populationSize) {
      // Tournament selection for parents
      const parent1 = tournamentSelection(population, fitnessScores, config.tournamentSize);
      const parent2 = tournamentSelection(population, fitnessScores, config.tournamentSize);

      // Crossover
      let offspring: PromptGenome;
      if (Math.random() < config.crossoverRate) {
        offspring = uniformCrossover(parent1, parent2);
      } else {
        offspring = Math.random() < 0.5 ? parent1 : parent2;
      }

      // Mutation
      offspring = mutate(offspring, config.mutationRate);

      nextGeneration.push(offspring);
    }

    // 7. Replace population
    population = nextGeneration;
  }

  // 8. Return best from final generation
  const finalFitnessScores = await Promise.all(
    population.map(async g => {
      const prompt = genomeToPrompt(g);
      return await fitnessFunction(prompt, testCases);
    })
  );

  const bestIdx = finalFitnessScores.indexOf(Math.max(...finalFitnessScores));
  return population[bestIdx];
}
```

### Evolution Example

```markdown
## Evolution of "Implement Authentication" Prompt

Generation 0 (baseline):
Population: 20 random variations of baseline
Best fitness: 0.45
Best prompt: [verbose prose version]
Tokens: 150

Generation 5:
Best fitness: 0.62 (+38%)
Best prompt: [structured list format emerging]
Tokens: 95 (-37%)

Generation 10:
Best fitness: 0.74 (+64%)
Best prompt: [XML tags with examples]
Tokens: 68 (-55%)

Generation 20:
Best fitness: 0.85 (+89%)
Best prompt: [Compact XML + math notation]
Tokens: 48 (-68%)

Generation 35:
Best fitness: 0.91 (+102%)
Best prompt: [Highly optimized, minimal tokens]
Tokens: 42 (-72%)

Generation 40-50:
Best fitness: 0.91 (converged)
No further improvement
Final: 72% token reduction, quality maintained
```

---

## Application to Our Project

### Phase 1: CLAUDE.md Optimization

**Goal**: Evolve CLAUDE.md to optimal form.

```typescript
// experiments/runner/evolve-claudemd.ts

interface CLAUDEmdGenome {
  sections: {
    stack: SectionGenome;
    principles: SectionGenome;
    workflow: SectionGenome;
    rules: SectionGenome;
  };
  overallFormat: 'prose' | 'structured' | 'mixed';
  abbreviationsUsed: string[];
}

interface SectionGenome {
  format: 'prose' | 'list' | 'table' | 'xml' | 'math';
  detailLevel: 'high' | 'medium' | 'low';
  present: boolean;  // Should this section exist?
}

async function evolveClaudeMd() {
  const baseline = readFileSync('.claude/CLAUDE.md', 'utf-8');
  const testCases = loadStandardTaskSuite();  // 20 diverse tasks

  const config: GAConfig = {
    populationSize: 15,
    generations: 30,
    mutationRate: 0.15,
    crossoverRate: 0.7,
    eliteCount: 2,
    tournamentSize: 3
  };

  const optimized = await geneticAlgorithmOptimization(
    baseline,
    testCases,
    config
  );

  // Write optimized version
  const optimizedPrompt = genomeToPrompt(optimized);
  writeFileSync('.claude/CLAUDE-optimized.md', optimizedPrompt);

  // Compare
  console.log('Baseline tokens:', countTokens(baseline));
  console.log('Optimized tokens:', countTokens(optimizedPrompt));
  console.log('Reduction:', calculateReduction(baseline, optimizedPrompt));

  // Validate quality maintained
  await validateQuality(baseline, optimizedPrompt, testCases);
}
```

### Phase 2: Skill Optimization

**Goal**: Evolve each Skill independently.

```typescript
// experiments/runner/evolve-skill.ts

async function evolveSkill(skillName: string) {
  const skill = loadSkill(skillName);
  const testCases = loadSkillTestCases(skillName);  // Domain-specific tests

  // Skill-specific fitness function
  const skillFitness = (prompt: string, tests: TestCase[]) => {
    return (
      0.5 * taskSuccessRate(prompt, tests) +
      0.3 * tokenEfficiency(prompt) +
      0.2 * adherenceRate(prompt, tests)
    );
  };

  const config: GAConfig = {
    populationSize: 12,
    generations: 25,
    mutationRate: 0.12,
    crossoverRate: 0.7,
    eliteCount: 1,
    tournamentSize: 3
  };

  const optimized = await geneticAlgorithmOptimization(
    skill.content,
    testCases,
    config
  );

  // Update skill
  await updateSkill(skillName, genomeToPrompt(optimized));

  // Log to .ai-knowledge
  await logOptimization({
    type: 'skill',
    name: skillName,
    beforeTokens: countTokens(skill.content),
    afterTokens: countTokens(genomeToPrompt(optimized)),
    qualityChange: await measureQualityChange(skill, optimized, testCases),
    generations: config.generations,
    finalFitness: await skillFitness(genomeToPrompt(optimized), testCases)
  });
}
```

### Phase 3: Multi-Objective Optimization

**Goal**: Optimize for BOTH quality AND token efficiency simultaneously.

```typescript
// Multi-objective fitness function

interface MultiObjectiveFitness {
  quality: number;      // 0-1
  efficiency: number;   // 0-1
  dominates: (other: MultiObjectiveFitness) => boolean;
}

function multiObjectiveFitness(
  prompt: string,
  testCases: TestCase[]
): MultiObjectiveFitness {
  const quality = measureQuality(prompt, testCases);
  const tokens = countTokens(prompt);
  const efficiency = 1.0 - (tokens / 1000);  // Normalize

  return {
    quality,
    efficiency,
    dominates: function(other: MultiObjectiveFitness): boolean {
      // Pareto dominance
      return (
        this.quality >= other.quality &&
        this.efficiency >= other.efficiency &&
        (this.quality > other.quality || this.efficiency > other.efficiency)
      );
    }
  };
}

function paretoSelection(
  population: PromptGenome[],
  fitnessScores: MultiObjectiveFitness[]
): PromptGenome[] {
  // Return Pareto front (non-dominated solutions)
  const paretoFront: PromptGenome[] = [];

  for (let i = 0; i < population.length; i++) {
    const genome = population[i];
    const fitness = fitnessScores[i];

    let dominated = false;
    for (let j = 0; j < population.length; j++) {
      if (i !== j && fitnessScores[j].dominates(fitness)) {
        dominated = true;
        break;
      }
    }

    if (!dominated) {
      paretoFront.push(genome);
    }
  }

  return paretoFront;
}
```

**Example Pareto front**:
```markdown
Non-dominated solutions (all on Pareto front):

Solution A: Quality 0.95, Efficiency 0.30 (350 tokens)
  → Highest quality, moderate tokens

Solution B: Quality 0.90, Efficiency 0.60 (200 tokens)
  → Balanced quality and efficiency

Solution C: Quality 0.82, Efficiency 0.85 (100 tokens)
  → Maximum efficiency, acceptable quality

All three are Pareto optimal (can't improve one without hurting the other)
Choose based on priorities: Quality-first? Pick A. Balanced? Pick B. Minimize cost? Pick C.
```

---

## Advanced Techniques

### 1. Island Model (Parallel Evolution)

**Concept**: Multiple populations evolve independently, occasionally exchange best solutions.

```typescript
interface Island {
  population: PromptGenome[];
  fitnessScores: number[];
  bestGenome: PromptGenome;
}

async function islandModelGA(
  initialPrompt: string,
  testCases: TestCase[],
  islandCount: number = 4,
  migrationInterval: number = 5
) {
  // Initialize islands
  const islands: Island[] = [];
  for (let i = 0; i < islandCount; i++) {
    islands.push({
      population: initializePopulation(initialPrompt, populationSize = 10),
      fitnessScores: [],
      bestGenome: null
    });
  }

  // Evolve islands in parallel
  for (let gen = 0; gen < totalGenerations; gen++) {
    // Evolve each island independently
    await Promise.all(islands.map(island => evolveIsland(island, testCases)));

    // Migration every N generations
    if (gen % migrationInterval === 0) {
      migrateIndividuals(islands, migrationCount = 2);
    }
  }

  // Return best across all islands
  return findGlobalBest(islands);
}

function migrateIndividuals(islands: Island[], count: number) {
  for (let i = 0; i < islands.length; i++) {
    const sourceIsland = islands[i];
    const targetIsland = islands[(i + 1) % islands.length];

    // Send best individuals to next island
    const migrants = selectBest(sourceIsland, count);
    targetIsland.population.push(...migrants);

    // Remove worst to maintain population size
    targetIsland.population = removeWorst(targetIsland, count);
  }
}
```

**Benefits**:
- Parallel evolution explores solution space more thoroughly
- Migration prevents premature convergence
- Different islands may find different local optima
- Faster overall (parallelizable)

### 2. Adaptive Mutation Rate

**Concept**: Adjust mutation rate based on population diversity.

```typescript
function calculatePopulationDiversity(population: PromptGenome[]): number {
  // Measure how different genomes are from each other
  let totalDistance = 0;
  let comparisons = 0;

  for (let i = 0; i < population.length; i++) {
    for (let j = i + 1; j < population.length; j++) {
      totalDistance += genomeDistance(population[i], population[j]);
      comparisons++;
    }
  }

  return totalDistance / comparisons;  // Average distance
}

function adaptiveMutationRate(
  population: PromptGenome[],
  baseMutationRate: number = 0.1
): number {
  const diversity = calculatePopulationDiversity(population);

  if (diversity < 0.2) {
    // Low diversity → increase mutation to explore more
    return baseMutationRate * 2.0;
  } else if (diversity > 0.8) {
    // High diversity → decrease mutation to converge
    return baseMutationRate * 0.5;
  } else {
    return baseMutationRate;
  }
}
```

### 3. Novelty Search

**Concept**: Reward novelty, not just fitness.

```typescript
interface NoveltyMetric {
  fitness: number;
  novelty: number;
  combined: number;
}

function noveltySearch(
  population: PromptGenome[],
  archive: PromptGenome[],  // All previously seen genomes
  fitnessScores: number[]
): NoveltyMetric[] {
  return population.map((genome, i) => {
    const fitness = fitnessScores[i];

    // Measure how different this genome is from archive
    const distances = archive.map(archiveGenome =>
      genomeDistance(genome, archiveGenome)
    );

    // Novelty = average distance to k-nearest neighbors
    const kNearest = 15;
    const nearestDistances = distances.sort().slice(0, kNearest);
    const novelty = nearestDistances.reduce((a, b) => a + b) / kNearest;

    // Combined score: fitness + novelty
    const combined = 0.7 * fitness + 0.3 * novelty;

    return { fitness, novelty, combined };
  });
}
```

**Why this helps**:
- Prevents premature convergence to local optimum
- Explores unusual solutions that might be better
- Encourages diversity in solution space

---

## Validation Experiments

### Experiment 1: GA vs Manual Optimization

**Hypothesis**: Genetic algorithm finds better optimization than manual refinement.

**Method**:
1. Baseline: Current CLAUDE.md (manual optimization)
2. Treatment: GA-optimized CLAUDE.md (30 generations)
3. Run 30 tasks with each
4. Measure: success rate, quality, tokens, time

**Success criteria**:
- ✅ GA achieves ≥ 10% better token efficiency than manual
- ✅ Quality maintained or improved
- ✅ GA finds non-obvious optimizations

### Experiment 2: Convergence Speed

**Hypothesis**: GA converges to good solution within 20 generations.

**Method**:
1. Run GA for 50 generations
2. Track fitness at each generation
3. Identify convergence point (no improvement for 5 gens)
4. Validate quality of solution at convergence

**Success criteria**:
- ✅ Convergence within 20 generations
- ✅ Converged solution is ≥ 90% fitness
- ✅ No quality degradation at convergence

### Experiment 3: Multi-Objective Optimization

**Hypothesis**: Pareto front provides better quality-efficiency trade-offs than single-objective.

**Method**:
1. Run single-objective GA (fitness = quality only)
2. Run multi-objective GA (quality + efficiency)
3. Compare Pareto front solutions to single-objective best
4. Measure: Are Pareto solutions truly non-dominated?

**Success criteria**:
- ✅ Pareto front includes solutions dominating single-objective best
- ✅ Pareto front offers meaningful trade-offs
- ✅ Can select solution matching our priority (quality vs efficiency)

### Experiment 4: Island Model Benefit

**Hypothesis**: Island model finds better solutions than single population.

**Method**:
1. Control: Single population, 40 individuals, 30 generations
2. Treatment: 4 islands, 10 individuals each, 30 generations, migrate every 5
3. Same total compute (400 evaluations each)
4. Compare best solution quality

**Success criteria**:
- ✅ Island model finds better or equal solution
- ✅ Island model has higher population diversity
- ✅ Implementation is parallelizable (faster wall-clock time)

---

## Practical Implementation Guide

### Week 1: Foundation

```typescript
// Step 1: Define genome structure
interface PromptGenome {
  // ... (as defined earlier)
}

// Step 2: Implement genome ↔ prompt conversion
function genomeToPrompt(genome: PromptGenome): string {
  // Convert genome to actual CLAUDE.md text
}

function parsePromptToGenome(prompt: string): PromptGenome {
  // Parse CLAUDE.md into genome structure
}

// Step 3: Create fitness function
async function fitnessFunction(
  prompt: string,
  testCases: TestCase[]
): Promise<number> {
  // Run test cases, measure success
}

// Step 4: Test on small example
const testGenome: PromptGenome = { /* ... */ };
const testPrompt = genomeToPrompt(testGenome);
const testFitness = await fitnessFunction(testPrompt, testCases);
console.log(`Test fitness: ${testFitness}`);
```

### Week 2: Basic GA

```typescript
// Step 5: Implement genetic operators
function mutate(genome: PromptGenome, rate: number): PromptGenome { /* ... */ }
function crossover(p1: PromptGenome, p2: PromptGenome): PromptGenome { /* ... */ }
function select(population: PromptGenome[], fitness: number[]): PromptGenome { /* ... */ }

// Step 6: Implement basic GA loop
async function basicGA(
  initialPrompt: string,
  testCases: TestCase[],
  generations: number = 10
): Promise<PromptGenome> {
  let population = initializePopulation(initialPrompt, 10);

  for (let gen = 0; gen < generations; gen++) {
    const fitness = await evaluatePopulation(population, testCases);
    const nextGen = createNextGeneration(population, fitness);
    population = nextGen;
  }

  return getBest(population);
}

// Step 7: Run on small section of CLAUDE.md
const stackSection = extractSection('.claude/CLAUDE.md', 'Stack');
const optimizedStack = await basicGA(stackSection, testCases, 10);
console.log('Before:', stackSection);
console.log('After:', genomeToPrompt(optimizedStack));
```

### Week 3: Full System

```typescript
// Step 8: Expand to full CLAUDE.md
const fullClaude = readFileSync('.claude/CLAUDE.md', 'utf-8');
const optimized = await basicGA(fullClaude, fullTestSuite, 30);

// Step 9: Validate quality
const qualityMaintained = await validateQuality(fullClaude, optimized, testCases);
if (qualityMaintained) {
  writeFileSync('.claude/CLAUDE-ga-optimized.md', genomeToPrompt(optimized));
  console.log('GA optimization successful!');
}

// Step 10: Document results
await logOptimization({
  method: 'genetic-algorithm',
  beforeTokens: countTokens(fullClaude),
  afterTokens: countTokens(genomeToPrompt(optimized)),
  generations: 30,
  populationSize: 15,
  finalFitness: await fitnessFunction(genomeToPrompt(optimized), testCases),
  qualityDelta: await measureQualityChange(fullClaude, optimized),
  timestamp: new Date().toISOString()
});
```

### Week 4: Advanced Features

```typescript
// Step 11: Implement island model
const islandOptimized = await islandModelGA(fullClaude, testCases, 4, 5);

// Step 12: Implement multi-objective
const paretoFront = await multiObjectiveGA(fullClaude, testCases);
console.log('Pareto front solutions:');
paretoFront.forEach((solution, i) => {
  console.log(`Solution ${i}:`, {
    quality: solution.quality,
    efficiency: solution.efficiency,
    tokens: countTokens(genomeToPrompt(solution.genome))
  });
});

// Step 13: Let user choose from Pareto front
const chosenSolution = await promptUser('Which solution do you prefer?', paretoFront);
writeFileSync('.claude/CLAUDE-final.md', genomeToPrompt(chosenSolution.genome));
```

---

## Key Takeaways

1. **Prompts can evolve** - Treat them as genomes that improve through natural selection
2. **Fitness is multi-dimensional** - Quality, efficiency, clarity all matter
3. **Mutation explores** - Small changes discover better variations
4. **Crossover exploits** - Combine successful patterns
5. **Selection guides** - Keep the fittest, discard the weak

**For our project**:
- ✅ GA can automate CLAUDE.md optimization
- ✅ Finds non-obvious optimizations humans miss
- ✅ Multi-objective optimization handles quality vs efficiency
- ✅ Island model explores solution space thoroughly
- ✅ Convergence typically within 20-30 generations

**Caution**: GA requires significant compute (hundreds of evaluations). Start with small sections before optimizing full CLAUDE.md.

---

**Document Version**: 1.0
**Last Updated**: 2025-11-03
**Status**: Ready for experimentation
**Related Docs**: README.md, learning-dspy-optimization.md, pattern-continuous-optimization.md
