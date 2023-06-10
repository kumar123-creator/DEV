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

    const gridData = jsonData.map(item => ({
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
          title: "Edit Row",
        },
        onRowUpdating: async e => {
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
        onRowRemoving: async e => {
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
  }

  .dx-popup-title {
    font-size: 18px;
    font-weight: bold;
    margin: 0;
  }

  .dx-toolbar-label {
    font-weight: bold;
  }
</style>

<div id="dataGrid"></div>
