<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  // One report, one problem. The desk's one-day notes split off from `parser`: a day off is not a month flip.
  const problemOf = (r: Report) => (notes.oneDayMove.includes(r.id) ? 'one-day' : causeOf(r, tenantOf(r)) ?? 'unplaced');

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const rows = $derived(
    (['parser', 'hint', 'one-day', 'unplaced'] as const)
      .map((problem) => ({ problem, n: merged.filter((r) => problemOf(r) === problem).length }))
      .filter((row) => row.n > 0),
  );
  const miscounted = $derived(merged.filter((r) => problemOf(r) !== 'one-day' && r.meta.intakeTag !== problemOf(r)).length);
</script>

<table>
  <thead><tr><th>problem</th><th>reports</th></tr></thead>
  <tbody>
    {#each rows as row (row.problem)}
      <tr><td><code>{row.problem}</code></td><td>{row.n}</td></tr>
    {/each}
    <tr><td><strong>total</strong></td><td><strong>{merged.length}</strong></td></tr>
  </tbody>
</table>

<p class="note">Intake tagged <strong>{miscounted}</strong> of these reports with the wrong cause.</p>

<style>
  .note { color: var(--muted); font-size: 0.85em; }
</style>
