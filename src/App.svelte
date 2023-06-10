import React, { useEffect } from "react";
import DataGrid, { Column, Editing, Paging, Pager, Popup } from "devextreme-react/data-grid";
import "devextreme/dist/css/dx.common.css";
import "devextreme/dist/css/dx.light.css";

const App = () => {
  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(
          "https://api.recruitly.io/api/candidate?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E"
        );
        const responseData = await response.json();
        const gridData = responseData.data.map((item) => ({
          id: item.id,
          firstName: item.firstName,
          surname: item.surname,
          email: item.email,
          mobile: item.mobile,
        }));
        setGridDataSource(gridData);
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    };

    fetchData();
  }, []);

  const onRowUpdating = async (e) => {
    const updatedData = e.newData;
    try {
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
        dataGridInstance.current.instance.refresh();
      } else {
        console.error("Error updating data:", response.statusText);
      }
    } catch (error) {
      console.error("Error updating data:", error);
    }
  };

  const onRowRemoving = async (e) => {
    const removedData = e.data;
    try {
      const response = await fetch(
        `https://api.recruitly.io/api/candidate/${removedData.id}?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E`,
        {
          method: "DELETE",
        }
      );

      if (response.ok) {
        dataGridInstance.current.instance.refresh();
      } else {
        console.error("Error deleting data:", response.statusText);
      }
    } catch (error) {
      console.error("Error deleting data:", error);
    }
  };

  const dataGridInstance = React.useRef();

  const [gridDataSource, setGridDataSource] = React.useState([]);

  return (
    <div id="dataGrid">
      <DataGrid
        ref={dataGridInstance}
        dataSource={gridDataSource}
        showBorders={true}
        remoteOperations={true}
      >
        <Editing
          mode="popup"
          allowDeleting={true}
          allowAdding={true}
          allowUpdating={true}
          onRowUpdating={onRowUpdating}
          onRowRemoving={onRowRemoving}
        >
          <Popup title="Row in the editing state" showTitle={true} />
        </Editing>
        <Column dataField="id" caption="ID" />
        <Column dataField="firstName" caption="First Name" />
        <Column dataField="surname" caption="Surname" />
        <Column dataField="email" caption="Email" />
        <Column dataField="mobile" caption="Mobile" />
        <Paging pageSize={10} />
        <Pager
          showPageSizeSelector={true}
          allowedPageSizes={[5, 10, 20]}
          showInfo={true}
        />
      </DataGrid>
    </div>
  );
};

export default App;
