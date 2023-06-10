<script>
  import { onMount } from "svelte";
  import DevExpress from "devextreme";

  // Sample data
  let jsonData = [];
  let data = [];

  onMount(async () => {
    const response = await fetch(
      "https://api.recruitly.io/api/candidate?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E"
    );

    const responseData = await response.json();
    jsonData = responseData.data;

    const gridData = jsonData.map((item) => ({
      id: item.id,
      firstName: item.firstName,
      surname: item.surname,
      email: item.email,
      mobile: item.mobile,
    }));

    var dataGrid = new DevExpress.ui.dxDataGrid("#dataGrid", {
      dataSource: gridData,
      columns: [
        { dataField: "id", caption: "ID" },
        { dataField: "firstName", caption: "First Name" },
        { dataField: "surname", caption: "Surname" },
        { dataField: "email", caption: "Email" },
        { dataField: "mobile", caption: "Mobile" },
      ],
      showBorders: true,
      filterRow: {
        visible: true,
      },
      editing: {
        allowDeleting: true,
        allowAdding: true,
        allowUpdating: true,
        mode: "popup",
        form: {
          labelLocation: "top",
        },
        popup: {
          showTitle: true,
          title: "Row in the editing state",
        },
        onRowUpdating: async (e) => {
          const updatedData = e.newData;
          const response = await fetch(
            `https://api.recruitly.io/api/candidate?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E`,
            {
              method: "POST",
              headers: {
                "Content-Type": "application/json",
              },
              body: JSON.stringify(updatedData),
            }
          );

          if (response.ok) {
            e.cancel = true;
            dataGrid.refresh();
          } else {
            console.error("Error updating data:", response.statusText);
          }
        },
        onRowRemoving: async (e) => {
          const removedData = e.data;
          const response = await fetch(
            `https://api.recruitly.io/api/candidate/${removedData.id}?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E`,
            {
              method: "DELETE",
            }
          );

          if (response.ok) {
            dataGrid.refresh();
          } else {
            console.error("Error deleting data:", response.statusText);
          }
        },
      },
      paging: {
        pageSize: 10,
      },
      pager: {
        showPageSizeSelector: true,
        allowedPageSizes: [5, 10, 20],
        showInfo: true,
      },
    });
  });
</script>

<style>
  #dataGrid {
    height: 400px;
    font-family: Arial, sans-serif;
  }

  .dx-datagrid.dx-datagrid-rowsview {
    background-color: #f5f5f5;
  }

  .dx-datagrid.dx-datagrid-rowsview table {
    width: 100%;
    border-collapse: collapse;
  }

  .dx-datagrid.dx-datagrid-rowsview td,
  .dx-datagrid.dx-datagrid-rowsview th {
    padding: 8px;
    border: 1px solid #ddd;
    text-align: left;
  }

  .dx-datagrid.dx-datagrid-rowsview th {
    background-color: #eee;
  }

  .dx-datagrid.dx-datagrid-rowsview .dx-editor-cell .dx-texteditor-input {
    padding: 6px;
    border: 1px solid #ddd;
    border-radius: 4px;
    background-color: #fff;
    font-size: 14px;
    color: #333;
    box-sizing: border-box;
  }

  .dx-datagrid.dx-datagrid-rowsview .dx-filter-row .dx-editor-cell .dx-texteditor-input {
    padding: 4px;
  }

  .dx-datagrid.dx-datagrid-rowsview .dx-command-edit {
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .dx-datagrid.dx-datagrid-rowsview .dx-command-edit > a {
    margin: 0 5px;
  }

  .dx-datagrid.dx-datagrid-rowsview .dx-command-edit > a.dx-link {
    color: #0078d4;
    text-decoration: none;
  }

  .dx-datagrid.dx-datagrid-rowsview .dx-command-edit > a.dx-link:hover {
    text-decoration: underline;
  }

  .dx-datagrid.dx-datagrid-rowsview .dx-button {
    padding: 4px 8px;
    margin: 0;
    font-size: 14px;
  }

  .dx-datagrid.dx-datagrid-rowsview .dx-popup-title {
    font-size: 16px;
    font-weight: bold;
    color: #333;
  }
</style>

<div id="dataGrid"></div>
