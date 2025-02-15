<script lang="ts">
  import { page } from '$app/state';
  import FormLabel from '$lib/components/ui-framework/Form/shared/FormLabel.svelte';
  import Card from '$lib/components/ui-framework/Layout/Card.svelte';
  import { getMoment } from '$lib/helpers/time';
  import { useBudgetStore } from '$lib/stores/budget/budget.svelte';
  import { useExpenseStore } from '$lib/stores/expense/expense.svelte';
  import { useTripsStore } from '$lib/stores/trips/trips.svelte';
  import FormattedCurrency from '../FormattedCurrency.svelte';
  import AnchorButton from '../ui-framework/Form/AnchorButton.svelte';

  const id = page.params.id;
  const targetTrip = $derived(useTripsStore.data.find((item) => item._id === id));
  const budgets = $derived(useBudgetStore.data.filter((item) => item.tripId === id));
  const totalBudget = $derived(
    budgets.map((item) => item.amount || 0).reduce((partialSum, a) => partialSum + a, 0),
  );

  const expenses = $derived(useExpenseStore.data.filter((item) => item.tripId === id));

  const expensesFiltered = $derived(expenses.filter((item) => item.tripId === id && item.budgetId));
  const totalExpenses = $derived(
    expenses.map((item) => item.amount || 0).reduce((partialSum, a) => partialSum + a, 0),
  );

  const totalExpensesFiltred = $derived(
    expensesFiltered.map((item) => item.amount || 0).reduce((partialSum, a) => partialSum + a, 0),
  );
</script>

<div class="TripStats">
  <Card>
    <ul>
      <li>
        <div class="TotalExpense">
          <div>
            <FormLabel label="Total Expense" />
            <b><FormattedCurrency value={totalExpenses} /></b>
          </div>
          <AnchorButton href={`/trips/${id}/stats`}>Stats</AnchorButton>
        </div>
      </li>
      <li>
        <div class="StatsLabel">Budget: Total</div>
        <div class="StatsValue">
          <a href={`/trips/${id}/budget`}>
            {#if budgets.length}
              <FormattedCurrency value={totalBudget} />
            {:else}
              Add budget
            {/if}
          </a>
        </div>
      </li>
      <li>
        <div class="StatsLabel">Budget: Remaining</div>
        <div class="StatsValue">
          <a href={`/trips/${id}/budget`}>
            {#if budgets.length}
              <u><FormattedCurrency value={totalBudget - totalExpensesFiltred} /></u>
            {/if}
          </a>
        </div>
      </li>
      <li>
        <div class="StatsLabel">Date</div>
        <div class="StatsValue">
          {getMoment(targetTrip?.startDate).format('D MMM YYYY')} - {getMoment(
            targetTrip?.endDate,
          ).format('D MMM YYYY')}
        </div>
      </li>
    </ul>
  </Card>
</div>

<style lang="scss">
  .TripStats {
    ul {
      display: block;
      margin: 0;
      padding: 0;
    }

    li {
      margin: 0;
      padding: 16px;
      display: flex;
      justify-content: space-between;
      font-size: 1.05rem;
      border-bottom: 1px solid var(--color-grey-400);
      padding-bottom: 16px;

      &:last-child {
        margin-bottom: 0;
        border-bottom: 0;
      }
    }

    .TotalExpense {
      display: flex;
      justify-content: space-between;
      flex-wrap: wrap;
      align-items: center;
      width: 100%;

      b {
        font-size: 1.5rem;
        font-weight: 500;
      }
    }

    :global(.Card) {
      padding: 0;
    }

    .StatsValue {
      font-weight: 600;
      a {
        text-decoration: none;
        color: var(--color-primary-800);
      }

      u {
        color: var(--color-danger-800);
        text-decoration: none;
      }
    }
  }
</style>
