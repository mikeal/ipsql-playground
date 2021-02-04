<script>
  import Paper, {Title, Subtitle, Content} from '@smui/paper'
  import Button, {Label} from '@smui/button'
  import Textfield from '@smui/textfield'
  import DataTable, {Head, Body, Row, Cell} from '@smui/data-table'
  import List, {Item, Text} from '@smui/list'
  import IPSQL from 'ipsql/memory'
  console.log(IPSQL)
  let ipsql = IPSQL.headless()
  export let statements = [
    'CREATE TABLE playground (id INT, text: VARCHAR(256))',
    'INSERT INTO playground VALUES (10, "hello world"), (20, "world hello")',
    'SELECT * from playground'
  ]
  export let sqlStatement = ''
  let transactions = []
  let db = null
  let error
  const run = async () => {
    if (!statements.length) return
    const sql = statements.shift()
    const proof = await ipsql.transaction({ db, sql })
    let { writes } = proof.value
    if (writes && writes.length) writes = writes.length
    if (writes.asCID === writes) writes = (await ipsql.getBlock(writes)).value.length
    let result = null
    if (proof.value.result) {
      result = (await ipsql.getBlock(proof.value.result)).value
    }
    transactions.push({ sql, db, writes: writes ? writes : 0, result })
    transactions = transactions
    db = proof.value.db
    run().catch(e => { error = e })
  }
  run().catch(e => { error = e })
  let _runSQL = async () => {
    const sql = sqlStatement
    sqlStatement = ''
    statements.push(sql)
    return run()
  }
  let runSQL = () => {
    error = null
    _runSQL().catch(e => { error = e })
  }
</script>

<style>
error {
  color: red;
  font-weight: bold;
}
</style>

<ipsql-docs>
  <DataTable table$aria-label="transactions">
    <Body>
    {#each transactions as trans}
      <Row>
        <Cell><code>{ trans.sql }</code> on { trans.db ? trans.db.toString().slice(0, 12) + '...' : null }</Cell>
        <Cell>(BlockWrites: { trans.writes })</Cell>
      </Row>
      {#if trans.result}
      <Row>
        <DataTable table$aria-label="results">
          <Body>
            {#each trans.result as row}
            <Row>
              {#each row as column}
              <Cell>{ column }</Cell>
              {/each}
            </Row>
            {/each}
          </Body>
        </DataTable>
      </Row>
      {/if}
    {/each}
    </Body>
  </DataTable>
  {#if error}
  <error>
    <h3>{error.message}</h3>
    <pre>{error.stack}</pre>
  </error>
  {/if}

  <Paper square class="paper-demo">
    <Title>Execute SQL statement</Title>
    <Content>
      <Textfield fullwidth textarea bind:value={sqlStatement} label="Enter SQL Statement" />
      <Button on:click={runSQL}>
        <Label>Run</Label>
      </Button>
    </Content>
  </Paper>
  <Paper square>
    <Title>Supported Grammar</Title>
<Content>
<pre>
- CREATE
  - TABLE
    - [x] INTEGER
    - [x] VARCHAR(size)
    - [x] FLOAT
    - [ ] DATE
    - [ ] TIME
    - [ ] DATETIME
    - [ ] BLOB
    - [ ] TEXT
- [ ] ALTER TABLE
- INSERT
  - [x] ROWS `INSERT INTO table_name VALUES ( 'test' )`
  - [x] COLUMNS `INSERT INTO table_name ( column_name ) VALUES ( 'test' )`
- UPDATE
  - [x] UPDATE SET w/o WHERE
  - [x] UPDATE SET WHERE
- DELETE
  - [ ] DELETE w/o WHERE (deletes table)
  - [ ] DELETE WHERE
- SELECT
  - [x] * `SELECT * FROM table_name`
  - [x] COLUMNS `SELECT column1, column2 FROM table_name`
  - [ ] ROWID meta column
  - [ ] TOP
  - [x] MIN
  - [x] MAX
  - [x] COUNT
  - [x] AVG
  - [x] SUM
  - [ ] LIKE *pattern*
  - WHERE
    - [x] AND
    - [x] OR
    - [x] ASC, DESC
    - [x] COMPARISON OPERATORS
    - [ ] GROUP BY
    - [x] ORDER BY
    - [ ] BETWEEN
    - [ ] LIKE
    - [ ] IN
    - [ ] NOT
    - [ ] IS NOT NULL, IS NULL
  - [ ] JOIN
  - [x] ORDER BY
  - [ ] GROUP BY
  - [ ] HAVING
  - [ ] UNION

This project is still an alpha. Some complex expression may not work and it's currentl limited
to a single expression at a time.
</pre>
</Content>

  </Paper>

</ipsql-docs>
