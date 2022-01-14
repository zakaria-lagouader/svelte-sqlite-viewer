<script>
    export let headers = [];
    export let rows = [];
</script>


<div class="table-responsive">
    <table class="table table-bordered table-sm">
        <thead class="table-dark">
          <tr>
            {#each headers as header}
                <th scope="col">{header}</th>
            {/each}
          </tr>
        </thead>
        <tbody>
            {#each rows as row}
                <tr>    
                    {#each row as column}
                        <td>{column}</td>
                    {/each}
                </tr>
            {/each}
        </tbody>
    </table>
</div>