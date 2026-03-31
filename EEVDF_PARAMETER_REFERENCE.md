# EEVDF Parameter Reference

## Exact Values Changed in commit 21fbb2b734e2

### Latency Parameters

```c
// kernel/sched/fair.c line 81-95
OLD (CFS):
  unsigned int sysctl_sched_latency = 4000000ULL;
  unsigned int normalized_sysctl_sched_latency = 4000000ULL;

EEVDF-INSPIRED:
  unsigned int sysctl_sched_latency = 3000000ULL;  // 3ms instead of 4ms
  unsigned int normalized_sysctl_sched_latency = 3000000ULL;
  // Comment added: "EEVDF-inspired: Lower latency for better responsiveness and fairness"
  // Comment added: "(EEVDF tuning: 3ms * (1 + ilog(ncpus)), units: nanoseconds)"

MIN GRANULARITY (lines 119-126):
OLD:
  unsigned int sysctl_sched_min_granularity = 400000ULL;
  unsigned int normalized_sysctl_sched_min_granularity = 400000ULL;

EEVDF-INSPIRED:
  unsigned int sysctl_sched_min_granularity = 250000ULL;  // 250μs instead of 400μs
  unsigned int normalized_sysctl_sched_min_granularity = 250000ULL;
  // Comment added: "EEVDF-inspired: Finer granularity for better deadline adherence"
  // Comment added: "(EEVDF tuning: 0.25 msec * (1 + ilog(ncpus)), units: nanoseconds)"
```

### Wakeup Parameters

```c
// kernel/sched/fair.c line 144-156
OLD:
  unsigned int sysctl_sched_wakeup_granularity = 200000UL;
  unsigned int normalized_sysctl_sched_wakeup_granularity = 200000UL;
  unsigned int __read_mostly sysctl_sched_migration_cost = 250000UL;

EEVDF-INSPIRED:
  unsigned int sysctl_sched_wakeup_granularity = 100000UL;  // 100μs instead of 200μs
  unsigned int normalized_sysctl_sched_wakeup_granularity = 200000UL;  // Normalized unchanged
  unsigned int __read_mostly sysctl_sched_migration_cost = 150000UL;  // 150μs instead of 250μs
  // Comment added: "EEVDF-inspired: Lower wake-up granularity for immediate responsiveness"
  // Comment added: "EEVDF: Lower for faster migration"
  // Comment added: "(EEVDF tuning: 0.5 msec * (1 + ilog(ncpus)), units: nanoseconds)"
```

### Bandwidth Slice

```c
// kernel/sched/fair.c line 200-205
OLD:
  unsigned int sysctl_sched_cfs_bandwidth_slice = 5000UL;

EEVDF-INSPIRED:
  unsigned int sysctl_sched_cfs_bandwidth_slice = 3000UL;  // 3ms instead of 5ms
  // Comment added: "(EEVDF tuning: 3 msec, units: microseconds)"
```

### Capacity Margins Arrays

```c
// kernel/sched/fair.c line 213-220
OLD:
  unsigned int sysctl_sched_capacity_margin_up[MAX_MARGIN_LEVELS] = {
    [0 ... MAX_MARGIN_LEVELS-1] = 1313};  /* Upmigrate: 78% */
  unsigned int sysctl_sched_capacity_margin_down[MAX_MARGIN_LEVELS] = {
    [0 ... MAX_MARGIN_LEVELS-1] = 1707};  /* Downmigrate: 60% */
  unsigned int sched_capacity_margin_up[NR_CPUS] = {
    [0 ... NR_CPUS-1] = 1313};  /* ~22% margin */
  unsigned int sched_capacity_margin_down[NR_CPUS] = {
    [0 ... NR_CPUS-1] = 1707};  /* ~40% margin */

EEVDF-INSPIRED:
  unsigned int sysctl_sched_capacity_margin_up[MAX_MARGIN_LEVELS] = {
    [0 ... MAX_MARGIN_LEVELS-1] = 1280};  /* EEVDF: Aggressive upmigrate */
  unsigned int sysctl_sched_capacity_margin_down[MAX_MARGIN_LEVELS] = {
    [0 ... MAX_MARGIN_LEVELS-1] = 1536};  /* EEVDF: Tight downmigrate */
  unsigned int sched_capacity_margin_up[NR_CPUS] = {
    [0 ... NR_CPUS-1] = 1280};  /* ~20% margin */
  unsigned int sched_capacity_margin_down[NR_CPUS] = {
    [0 ... NR_CPUS-1] = 1536};  /* ~33% margin */
```

## Summary Table

| Parameter | Old Value | New Value | Reduction | Unit |
|-----------|-----------|-----------|-----------|------|
| sysctl_sched_latency | 4000000 | 3000000 | -25% | ns |
| sysctl_sched_min_granularity | 400000 | 250000 | -37.5% | ns |
| sysctl_sched_wakeup_granularity | 200000 | 100000 | -50% | ns |
| sysctl_sched_migration_cost | 250000 | 150000 | -40% | ns |
| sysctl_sched_cfs_bandwidth_slice | 5000 | 3000 | -40% | μs |
| capacity_margin_up | 1313 | 1280 | -2.5% | margin |
| capacity_margin_down | 1707 | 1536 | -10% | margin |

## How to Apply to Any CFS Kernel

1. **Locate** scheduler parameters section in `kernel/sched/fair.c`
2. **Apply** the pecentage reductions above:
   - 25% for main latency
   - 37.5% for min granularity
   - 50% for wakeup granularity
   - 40% for migration cost
   - 40% for bandwidth slice
3. **Test** and adjust based on workload

## Percent Reduces Applied

```percentages
Latency Window:        -25%  = 3/4 of original
Min Granularity:       -37.5% = 5/8 of original  
Wakeup Granularity:    -50%  = 1/2 of original
Migration Cost:        -40%  = 3/5 of original
Bandwidth Slice:       -40%  = 3/5 of original
Upmigrate Margin:      -2.5% (conservative, efficiency focus)
Downmigrate Margin:    -10%  (aggressive, perf core promotion)
```

## Architecture Impact

**For Heterogeneous CPUs** (big.LITTLE, ARM big.LITTLE):
- More aggressive upmigration to performance cores
- Tighter efficiency core handling
- Better latency for interactive tasks on performance cores

**For Homogeneous CPUs** (all same core type):
- Pure latency improvement
- Better interactivity
- Minimal energy impact (already saturated utilization)

## Commands to Check Current Values

```bash
# Check current scheduler parameters
cat /proc/sys/kernel/sched_latency_ns
cat /proc/sys/kernel/sched_min_granularity_ns
cat /proc/sys/kernel/sched_wakeup_granularity_ns
cat /proc/sys/kernel/sched_migration_cost_ns

# Temporary runtime adjustment (persists until reboot)
echo 3000000 > /proc/sys/kernel/sched_latency_ns
echo 250000 > /proc/sys/kernel/sched_min_granularity_ns
echo 100000 > /proc/sys/kernel/sched_wakeup_granularity_ns
echo 150000 > /proc/sys/kernel/sched_migration_cost_ns
```

## Tuning for Different Workloads

### For Maximum Latency Reduction:
Use EEVDF values (above)

### For More Conservative Approach:
Use 50% of EEVDF reduction:
- sysctl_sched_latency: 3500000 (instead of 3000000)
- sysctl_sched_min_granularity: 325000 (instead of 250000)
- sysctl_sched_wakeup_granularity: 150000 (instead of 100000)

### For Balanced Performance/Efficiency:
Keep granularity reductions, reduce latency reduction:
- sysctl_sched_latency: 3500000
- sysctl_sched_min_granularity: 250000 (keep aggressive)
- sysctl_sched_wakeup_granularity: 100000 (keep aggressive)
- capacity_margin_up: 1296 (conservative)
- capacity_margin_down: 1620 (conservative)
