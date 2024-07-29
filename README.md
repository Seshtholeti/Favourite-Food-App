import React, { useEffect, useState } from "react";
import Header from "./Header";
import img1 from "./img/img1.jpg";
import img2 from "./img/img2.jpg";
const columnStyle = {
  flex: 1,
  display: "flex",
  flexDirection: "column",
  marginTop: "10px",
  marginLeft: "10px",
  gap: "1px",
  justifyContent: "center",
  alignItems: "center",
};
const containerStyle = {
  display: "flex",
  alignItems: "flex-start",
  color: "#333",
  padding: "20px",
  height: "80.9vh",
  width: "100vw",
  boxSizing: "border-box",
  // overflow: "hidden",
};
const imageContainerStyle = {
  width: "80%",
  height: "90%",
  display: "flex",
  justifyContent: "center",
  alignItems: "center",
  borderRadius: "5px",
  marginLeft: "10px",
  marginTop: "25px",
};
const imageStyle = {
  width: "100%",
  height: "100%",
  borderRadius: "5px",
};
const dataContainerStyle = {
  display: "flex",
  flexDirection: "row",
  justifyContent: "space-between",
  width: "100%",
  height: "100%",
  marginLeft: "10px",
  fontSize: "50px",
  marginTop: "20px",
};
const cardStyle = {
  backgroundColor: "#820882",
  padding: "9px",
  borderRadius: "5px",
  display: "flex",
  justifyContent: "space-between",
  alignItems: "center",
  height: "40px",
  color: "#fff",
  fontSize: "28.2px",
  // width: "600px",
  width: "90%",
  fontWeight: "700",
  transition: "background-color 0.3s ease",
};
const hoveredCardStyle = {
  backgroundColor: "red",
  color: "white",
  cursor: "pointer",
};
const keyTranslations = {
  DEPARTMENT: "Leitung",
  CIQ: "Warteschleife",
  LWT: "Längste Wartezeit",
  OFFERED: "Gesamt",
  ANS: "Angenommen",
  ANS_RATE: "Angenommen %",
  RDY: "Bereit",
  TALK: "Im Gespräch",
  NOT_RDY: "Nicht bereit",
  ONLINE: "Angemeldet",
};
const orderedKeys = [
  "DEPARTMENT",
  "OFFERED",
  "ANS",
  "ANS_RATE",
  "CIQ",
  "LWT",
  "ONLINE",
  "RDY",
  "TALK",
  "NOT_RDY",
];
const App = () => {
  const [data, setData] = useState([]);
  const [currentImageIndex, setCurrentImageIndex] = useState(0);
  const [hoveredCardIndex, setHoveredCardIndex] = useState(null);
  const images = [img1, img2];
  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(
          "https://52t6tr8gt5.execute-api.eu-west-2.amazonaws.com/UAT"
        );
        const responseData = await response.json();
        console.log(responseData);
        setData(responseData.body.flat());
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    };
    fetchData();
    const interval = setInterval(fetchData, 10000);
    return () => clearInterval(interval);
  }, []);
  // useEffect(() => {
  //   const imageRotationInterval = setInterval(() => {
  //     setCurrentImageIndex((prevIndex) => (prevIndex + 1) % images.length);
  //   }, 60000); // Set interval to 1 minute
  //   return () => clearInterval(imageRotationInterval);
  // }, [images.length]);
  const renderCard = (label, value, index) => (
    <div
      key={index}
      style={{
        ...cardStyle,
        ...(hoveredCardIndex === index ? hoveredCardStyle : null),
      }}
      onMouseEnter={() => handleMouseEnter(index)}
      onMouseLeave={() => handleMouseLeave()}
    >
      <span>{label}:</span>
      <span>{value}</span>
    </div>
  );
  const handleMouseEnter = (index) => {
    setHoveredCardIndex(index);
  };
  const handleMouseLeave = () => {
    setHoveredCardIndex(null);
  };
  const renderColumn = (items, department) => {
    return (
      <div style={columnStyle}>
        {items.map((item, index) => {
          const orderedItems = orderedKeys.map((key) => ({
            key: keyTranslations[key] || key,
            value: item[key],
          }));
          return orderedItems.map((orderedItem, subIndex) =>
            renderCard(
              orderedItem.key,
              orderedItem.value,
              `${department}-${index}-${subIndex}`
            )
          );
        })}
      </div>
    );
  };
  const queryItems = data.filter((item) => item.DEPARTMENT === "Queries");
  const reservationItems = data.filter(
    (item) => item.DEPARTMENT === "Reservation"
  );
  return (
    <div style={containerStyle}>
      {/* <div style={imageContainerStyle}>
        {images.length > 0 ? (
          <img
            src={images[currentImageIndex]}
            alt="Carousel"
            style={imageStyle}
          />
        ) : (
          <span>Upload Image</span>
        )}
      </div> */}
      {data.length > 0 && (
        <div style={dataContainerStyle}>
          {renderColumn(reservationItems, "Reservation")}
          {renderColumn(queryItems, "Query")}
        </div>
      )}
    </div>
  );
};
export default App;


there is a new metric, which is LOST_CALLs in the api

this is the api response
{"statusCode":200,"body":[[{"ANS_RATE":0,"OFFERED":0,"LOST_CALLS":0,"ANS":0,"RDY":0,"ONLINE":0,"NOT_RDY":0,"TALK":0,"LWT":"0:0","CIQ":0,"DEPARTMENT":"Queries"}],[{"ANS_RATE":0,"OFFERED":0,"LOST_CALLS":0,"ANS":0,"RDY":0,"ONLINE":0,"NOT_RDY":0,"TALK":0,"LWT":"0:0","CIQ":0,"DEPARTMENT":"Reservation"}],[{"ANS_RATE":0,"OFFERED":0,"LOST_CALLS":0,"ANS":0,"RDY":0,"ONLINE":0,"NOT_RDY":0,"TALK":0,"LWT":"0:0","CIQ":0,"DEPARTMENT":"Group"}]]}

and LOST_CALLS cars should be in between LWT and online and german translation of LOST_CALS is Verpasst.

please do not change anything else
