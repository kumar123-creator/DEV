<script>
  import { onMount } from "svelte";
  import DevExpress from "devextreme";
  import "bootstrap/dist/css/bootstrap.min.css";

  let jsonData = [];
  let gridData = [];
  let dataGrid;

  onMount(async () => {
    const fetchData = async () => {
      try {
        const response = await fetch(
          "https://api.recruitly.io/api/candidate?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA"
        );
        const responseData = await response.json();
        jsonData = responseData.data;
        gridData = jsonData.map((item) => ({
          firstName: item.firstName,
          surName: item.surName,
          id: item.id,
          email: item.email,
          mobile: item.mobile,
        }));

        renderDataGrid();
      } catch (error) {
        console.error("Failed to fetch data:", error);
      }
    };

    fetchData();
  });

  const renderDataGrid = () => {
    dataGrid = new DevExpress.ui.dxDataGrid(document.getElementById("dataGrid"), {
      dataSource: gridData,
      columns: [
        { dataField: "firstName", caption: "First Name", width: 200 },
        { dataField: "surName", caption: "Sur Name", width: 200 },
        { dataField: "id", caption: "ID", width: 200 },
        { dataField: "email", caption: "Email", width: 200 },
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
          width: 600,
          height: 400,
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
        pageSize: 20,
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
            e.data.id = responseData.id;
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
        console.log("Data sent to API:", e.newdata);
        try {
          const response = await fetch(
            `https://api.recruitly.io/api/candidate/${e.key}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`,
            {
              method: "PUT",
              headers: {
                "Content-Type": "application/json",
              },
              body: JSON.stringify(e.newdata),
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
  };
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
