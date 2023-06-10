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
			title: "Row in the editing state",
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
  
  <div id="dataGrid"></div>
  
  <style>
	#dataGrid {
	  height: 400px;
	}
  </style>
  