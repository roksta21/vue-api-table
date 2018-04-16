# Datatable
```html
<vue-api-table :url="'/api/v1/customers'" :columns="columns"></vue-api-table>
```

## Columns
```javascript
columns: {
    'ID': 'id',
    'Name': 'name',
    'Phone Number': 'phone_number',
    'Running Balance': 'account.running_balance',
}
```