import React, { useEffect, useState } from "react";
const containerStyle = {
  display: "flex",
  alignItems: "flex-start",
  backgroundColor: "blue",
  color: "#333",
  padding: "20px",

  height: "88.9vh",
  boxSizing: "border-box",
  overflow: "hidden",
};
const imageContainerStyle = {
  width: "80%",
  height: "90%",
  display: "flex",
  justifyContent: "center",
  alignItems: "center",
  borderRadius: "5px",
};
const imageStyle = {
  width: "100%",
  height: "100%",
  borderRadius: "5px",

  // objectFit: "cover",
};
const dataContainerStyle = {
  flex: 1,
  display: "flex",
  flexDirection: "column",
  marginTop: "10px",
  marginLeft: "110px",
  gap: "12px",
  height: "90%",
  justifyContent: "center",
  alignItems: "center",
};
// const headerStyle = {
//   textAlign: "center",
//   marginBottom: "20px",
//   fontSize: "24px",
//   color: "white",
//   fontWeight: "bold",
// };
const cardStyle = {
  backgroundColor: "#003366",
  padding: "11px",
  borderRadius: "5px",
  display: "flex",
  justifyContent: "space-between",
  alignItems: "center",
  height: "30px",
  color: "#fff",
  fontSize: "14px",
  width: "200px",
  transition: "background-color 0.3s ease",

  ":hover": {
    backgroundColor: "red",
    color: "white",
  },
};

const App = () => {
  const [data, setData] = useState(null);
  const [imageUrl, setImageUrl] = useState("");
  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(
          "https://1mh8wqhvf7.execute-api.eu-west-2.amazonaws.com/reservation"
        );
        const data = await response.json();
        console.log("seshu", data.body.Item);
        let filtered = data.body.Item;
        delete filtered["DEPARTMENT"];
        console.log(filtered);
        setData(data.body.Item);
        console.log("seshu", data.body.Item);
        setImageUrl(
          "https://wb-quicksight-html.s3.eu-west-2.amazonaws.com/Whitbread-image.jpg"
        ); // Replace with the actual URL
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    };
    // Fetch data initially
    fetchData();
    // Fetch data every 10 seconds (adjust interval as needed)
    const interval = setInterval(fetchData, 10000);
    // Cleanup interval on component unmount
    return () => clearInterval(interval);
  }, []);

  const renderCard = (label, value) => (
    <div style={cardStyle}>
      <span>{label}:</span>
      <span>{value}</span>
    </div>
  );
  return (
    <div style={containerStyle}>
      <div style={imageContainerStyle}>
        {imageUrl ? (
          <img src={imageUrl} alt="Uploaded" style={imageStyle} />
        ) : (
          <span>Upload Image</span>
        )}
      </div>
      <div style={dataContainerStyle}>
        {/* <h2 style={headerStyle}>Reservation Centre</h2> */}
        {data && Object.keys(data).map((key) => renderCard(key, data[key]))}
      </div>
    </div>
  );
};
export default App;
