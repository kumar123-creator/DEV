<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];

  onMount(async () => {
    const response = await fetch(
      "https://api.recruitly.io/api/candidate?apiKey=TEST9349C0221517DA4942E39B5DF18C68CDA154"
    );
    const responseData = await response.json();
    jsonData = responseData.data;

    gridData = jsonData.map((item) => ({
      id: item.id,
      firstName: item.firstName,
      surname: item.surname,
      email: item.email,
      mobile: item.mobile,
    }));

    const dataGrid = new DevExpress.ui.dxDataGrid(document.getElementById("dataGrid"), {
      dataSource: gridData,
      columns: [
        { dataField: "id", caption: "ID", width: 150 },
        { dataField: "firstName", caption: "Full Name", width: 200 },
        { dataField: "surname", caption: "Surname", width: 200 },
        { dataField: "email", caption: "Email", width: 250 },
        { dataField: "mobile", caption: "Mobile", width: 150 },
        // Define other columns as needed
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
          width: "90%",
          height: "90%",
          position: {
            my: "center",
            at: "center",
            of: window,
          },
        },
        texts: {
          saveRowChanges: "Save",
          cancelRowChanges: "Cancel",
          deleteRow: "Delete",
          confirmDeleteMessage: "Are you sure you want to delete this record?",
        },
      },
      paging: {
        pageSize: 10,
      },
      onRowInserting: async (e) => {
        console.log("Data being sent to API:", e.data);
        try {
          const response = await fetch(
            "https://api.recruitly.io/api/candidate?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA",
            {
              method: "POST",
              headers: {
                "Content-Type": "application/json",
              },
              body: JSON.stringify(e.data),
            }
          );

          const responseData = await response.json();
          if (response.ok) {
            e.data.firstName = responseData.firstName;
            gridData.push(e.data);
            dataGrid.refresh();
          } else {
            console.error("Failed to add record:", responseData.error);
          }
        } catch (error) {
          console.error("Failed to add record:", error);
        }
      },
      onRowUpdating: async (e) => {
        try {
          console.log(e);
          var newData = {
            id: e.key.id,
            firstName: e.newData.firstName === undefined ? e.oldData.firstName : e.newData.firstName,
            surname: e.newData.surname === undefined ? e.oldData.surname : e.newData.surname,
            email: e.newData.email === undefined ? e.oldData.email : e.newData.email,
            mobile: e.newData.mobile === undefined ? e.oldData.mobile : e.newData.mobile,
          };

          console.log(newData);
          const response = await fetch(
            `https://api.recruitly.io/api/candidate/${e.key.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`,
            {
              method: "PUT",
              headers: {
                "Content-Type": "application/json",
              },
              body: JSON.stringify(newData),
            }
          );
          const responseData = await response.json();
          if (response.ok) {
            const updatedItemIndex = gridData.findIndex((item) => item.id === e.key);
            if (updatedItemIndex > -1) {
              // Replace the old item with the updated item from the response
              gridData[updatedItemIndex] = responseData;
              dataGrid.refresh();
            }
          } else {
            console.error("Failed to update record:", responseData.error);
          }
        } catch (error) {
          console.error("Failed to update record:", error);
        }
      },
      onRowRemoving: async (e) => {
        console.log("Data being sent to API:", e.data);
        try {
          const response = await fetch(
            `https://api.recruitly.io/api/candidate/${e.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`,
            {
              method: "DELETE",
            }
          );
          if (response.ok) {
            const removedItemIndex = gridData.findIndex((item) => item.id === e.key);
            if (removedItemIndex > -1) {
              gridData.splice(removedItemIndex, 1);
              dataGrid.refresh();
            }
          } else {
            console.error("Failed to delete record.");
          }
        } catch (error) {
          console.error("Failed to delete record:", error);
        }
      },
    });
  });
</script>

<style>
  body {
    background-color: #f4f7fa;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    margin: 0;
  }

  #dataGrid {
    width: 100%;
    height: 100%;
    background-color: #fff;
    border: 1px solid #ccc;
    border-radius: 4px;
    overflow: hidden;
    box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
  }

  .dx-popup-content {
    background-color: #fff;
    border: 1px solid #ccc;
    border-radius: 4px;
  }

  .dx-popup-title {
    background-color: #007bff;
    color: #fff;
    padding: 10px;
    font-weight: bold;
    border-bottom: 1px solid #ccc;
    border-top-left-radius: 4px;
    border-top-right-radius: 4px;
  }
</style>

<div id="dataGrid"></div>
