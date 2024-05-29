import React from "react";
function ReservationCenterHeader() {
  const footerStyle = {
    color: "white",
    // Dark blue color
    backgroundColor: "#800080",
    // Adjust font size as needed
    padding: "10px",
    height: "4.7vh",
    display: "flex",
    alignItems: "flex-start",
    fontSize: "24px",
  };
  return <div style={footerStyle}>LWT>>></div>;
}
export default ReservationCenterHeader;
this is my footer



import React, { useEffect, useState } from "react";
const containerStyle = {
  display: "flex",
  alignItems: "flex-start",
  // backgroundColor: "blue",
  color: "#333",
  padding: "20px",
  height: "80.9vh",
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
const cardStyle = {
  backgroundColor: "#800080",
  padding: "8px",
  borderRadius: "5px",
  display: "flex",
  justifyContent: "space-between",
  alignItems: "center",
  height: "30px",
  color: "#fff",
  fontSize: "14px",
  width: "200px",
  transition: "background-color 0.3s ease",
};
const hoveredCardStyle = {
  backgroundColor: "red",
  color: "white",
  pointer: "cursor",
};
const App = () => {
  const [data, setData] = useState(null);
  const [imageUrl, setImageUrl] = useState("");
  const [hoveredCards, setHoveredCards] = useState([]); // Track hovered state for each card
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
  const renderCard = (label, value, index) => (
    <div
      key={index}
      style={{
        ...cardStyle,
        ...(hoveredCards[index] ? hoveredCardStyle : null),
      }}
      onMouseEnter={() => handleMouseEnter(index)}
      onMouseLeave={() => handleMouseLeave(index)}
    >
      <span>{label}:</span>
      <span>{value}</span>
    </div>
  );
  const handleMouseEnter = (index) => {
    setHoveredCards((prev) => {
      const newHoveredCards = [...prev];
      newHoveredCards[index] = true;
      return newHoveredCards;
    });
  };
  const handleMouseLeave = (index) => {
    setHoveredCards((prev) => {
      const newHoveredCards = [...prev];
      newHoveredCards[index] = false;
      return newHoveredCards;
    });
  };
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
        {data &&
          Object.keys(data).map((key, index) =>
            renderCard(key, data[key], index)
          )}
      </div>
    </div>
  );
};
export default App;



this is my dashboard.js


here we have metric GUEST_RELS and RES_CENTRE:   these values should be in the footer not with the same metrics coming from the api rest all same place
