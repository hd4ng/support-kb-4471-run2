<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // Which repo issue holds each problem split out of the merged ticket.
  let { ticket, repo, issues }: { ticket: number; repo: string; issues: { parser: number; hint: number; oneDayMove: number } } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const count = (cause: string) => groups.find((g) => g.cause === cause)?.reports.length ?? 0;
  const tenants = (cause: string) => groups.find((g) => g.cause === cause)?.tenants ?? 0;
  const oneDay = $derived(groups.reduce((n, g) => n + readAs(g, notes.oneDayMove), 0));
  const url = (n: number) => `https://github.com/${repo}/issues/${n}`;
</script>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th><th>ticket</th></tr></thead>
  <tbody>
    <tr><td><code>parser</code>: day-first dates read month first</td><td>{count('parser')}</td><td>{tenants('parser')}</td><td><a href={url(issues.parser)}>#{issues.parser}</a></td></tr>
    <tr><td><code>hint</code>: booking-v3 shows MM/DD/YYYY to day-first tenants</td><td>{count('hint')}</td><td>{tenants('hint')}</td><td><a href={url(issues.hint)}>#{issues.hint}</a></td></tr>
    <tr><td>one-day move: the date shifts one day, not a month</td><td>{oneDay}</td><td></td><td><a href={url(issues.oneDayMove)}>#{issues.oneDayMove}</a></td></tr>
  </tbody>
</table>
<p class="note">{merged.length} reports merged into #{ticket}. The {oneDay} one-day moves are also inside the <code>parser</code> count, where the desk files them by tenant.</p>

<style>
  .note { color: var(--muted); font-size: 0.85em; }
</style>
