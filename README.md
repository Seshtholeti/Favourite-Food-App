// import React, { useState, useEffect } from "react";
// import { FontAwesomeIcon } from "@fortawesome/react-fontawesome";
// import { faSignOutAlt } from "@fortawesome/free-solid-svg-icons";
// function Header({ onLogout }) {
//   const [currentTime, setCurrentTime] = useState(new Date());
//   useEffect(() => {
//     const timer = setInterval(() => {
//       setCurrentTime(new Date());
//     }, 1000);
//     return () => clearInterval(timer);
//   }, []);
//   const headerStyle = {
//     color: "white",
//     backgroundColor: "#00008B",
//     padding: "10px",
//     height: "50px",
//     display: "flex",
//     alignItems: "center",
//     fontSize: "40px",

//     justifyContent: "space-between",
//   };
//   const formatDate = (date) => {
//     const day = String(date.getDate()).padStart(2, "0");
//     const month = String(date.getMonth() + 1).padStart(2, "0");
//     const year = date.getFullYear();
//     return `${day}.${month}.${year}`;
//   };
//   const formatTime = (date) => {
//     return date.toLocaleTimeString("en-GB"); // en-GB locale for 24-hour format
//   };
//   const iconStyle = {
//     marginLeft: "20px",
//     fontSize: "30px",
//     color: "white",
//     cursor: "pointer",
//   };
//   const iconHoverStyle = {
//     color: "#c9302c",
//   };
//   return (
//     <div style={headerStyle}>
//       <div style={{ paddingLeft: "60px", paddingBottom: "5px" }}>
//         Reservation, Queries and Group
//       </div>
//       <div style={{ paddingRight: "60px", paddingBottom: "5px" }}>
//         {formatDate(currentTime)}&nbsp;&nbsp;{formatTime(currentTime)}
//         <FontAwesomeIcon
//           icon={faSignOutAlt}
//           style={iconStyle}
//           data-tip="Sign out"
//           onMouseOver={(e) => (e.target.style.color = iconHoverStyle.color)}
//           onMouseOut={(e) => (e.target.style.color = iconStyle.color)}
//           onClick={onLogout}
//         />
//       </div>
//     </div>
//   );
// }
// export default Header;

// import React, { useState, useEffect } from "react";
// import { FontAwesomeIcon } from "@fortawesome/react-fontawesome";
// import { faSignOutAlt } from "@fortawesome/free-solid-svg-icons";
// function Header({ onLogout }) {
//   const [currentTime, setCurrentTime] = useState(new Date());
//   useEffect(() => {
//     const timer = setInterval(() => {
//       setCurrentTime(new Date());
//     }, 1000);
//     return () => clearInterval(timer);
//   }, []);
//   const headerStyle = {
//     color: "white",
//     backgroundColor: "#00008B",
//     padding: "10px",
//     height: "50px",
//     display: "flex",
//     alignItems: "center",
//     fontSize: "40px",
//     justifyContent: "space-between",
//   };
//   const formatDate = (date) => {
//     const day = String(date.getDate()).padStart(2, "0");
//     const month = String(date.getMonth() + 1).padStart(2, "0");
//     const year = date.getFullYear();
//     return `${day}.${month}.${year}`;
//   };
//   const formatTime = (date) => {
//     return date.toLocaleTimeString("en-GB"); // en-GB locale for 24-hour format
//   };
//   const iconStyle = {
//     marginLeft: "20px",
//     fontSize: "30px",
//     color: "white",
//     cursor: "pointer",
//   };
//   const iconHoverStyle = {
//     color: "#c9302c",
//   };
//   return (
//     <div style={headerStyle}>
//       <div style={{ paddingLeft: "60px", paddingBottom: "5px" }}>
//         Wallboard-Germany
//       </div>
//       <div style={{ paddingRight: "60px", paddingBottom: "5px" }}>
//         {formatDate(currentTime)}&nbsp;&nbsp;{formatTime(currentTime)}
//         <FontAwesomeIcon
//           icon={faSignOutAlt}
//           style={iconStyle}
//           onMouseOver={(e) =>
//             (e.currentTarget.style.color = iconHoverStyle.color)
//           }
//           onMouseOut={(e) => (e.currentTarget.style.color = iconStyle.color)}
//           onClick={onLogout}
//         />
//       </div>
//     </div>
//   );
// }
// export default Header;

import React, { useState, useEffect } from "react";
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome";
import { faSignOutAlt } from "@fortawesome/free-solid-svg-icons";
function Header({ onLogout }) {
  const [currentTime, setCurrentTime] = useState(new Date());
  const [isHovered, setIsHovered] = useState(false);
  useEffect(() => {
    const timer = setInterval(() => {
      setCurrentTime(new Date());
    }, 1000);
    return () => clearInterval(timer);
  }, []);
  const headerStyle = {
    color: "white",
    backgroundColor: "#820882",
    padding: "10px",
    height: "50px",
    display: "flex",
    alignItems: "center",
    fontSize: "40px",
    width: "98vw",
    justifyContent: "space-between",
  };
  const formatDate = (date) => {
    const day = String(date.getDate()).padStart(2, "0");
    const month = String(date.getMonth() + 1).padStart(2, "0");
    const year = date.getFullYear();
    return `${day}.${month}.${year}`;
  };
  const formatTime = (date) => {
    return date.toLocaleTimeString("en-GB");
  };
  const iconContainerStyle = {
    display: "flex",
    alignItems: "center",
    marginRight: "40px",
  };
  const iconStyle = {
    marginLeft: "20px",
    fontSize: "30px",
    color: "white",
    cursor: "pointer",
  };
  const iconHoverStyle = {
    color: "#c9302c",
  };
  const logoutTextStyle = {
    fontSize: "10px",
    // marginLeft: "5px",
    visibility: isHovered ? "visible" : "hidden",
  };
  return (
    <div style={headerStyle}>
      <div style={{ paddingLeft: "60px", paddingBottom: "5px", width: "100%" }}>
        Anrufstatistik
      </div>
      <div style={iconContainerStyle}>
        <div>
          {formatDate(currentTime)}&nbsp;&nbsp;{formatTime(currentTime)}
        </div>
        <div
          style={logoutTextStyle}
          onMouseOver={() => setIsHovered(true)}
          onMouseOut={() => setIsHovered(false)}
          onClick={onLogout}
        >
          Logout
        </div>
        <FontAwesomeIcon
          icon={faSignOutAlt}
          style={iconStyle}
          onMouseOver={(e) => {
            e.currentTarget.style.color = iconHoverStyle.color;
            setIsHovered(true);
          }}
          onMouseOut={(e) => {
            e.currentTarget.style.color = iconStyle.color;
            setIsHovered(false);
          }}
          onClick={onLogout}
        />
      </div>
    </div>
  );
}
export default Header;


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
  gap: "14px",
  justifyContent: "center",
  alignItems: "center",
};
const containerStyle = {
  display: "flex",
  alignItems: "flex-start",
  color: "#333",
  padding: "20px",
  height: "81.9vh",
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
  height: "90%",
  marginLeft: "10px",
  marginTop: "20px",
};
const cardStyle = {
  backgroundColor: "#820882",
  padding: "9px",
  borderRadius: "5px",
  display: "flex",
  justifyContent: "space-between",
  alignItems: "center",
  height: "60px",
  color: "#fff",
  fontSize: "15px",
  width: "500px",
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
this is dashboard .js above one

import React from "react";
import "./App.css";
function ReservationCenterHeader() {
  const footerStyle = {
    color: "white",
    // Dark blue color
    backgroundColor: "#820882",
    // Adjust font size as needed
    padding: "10px",
    height: "4.7vh",
    display: "flex",
    alignItems: "flex-start",
    fontSize: "24px",
  };

  const Marquee = ({ text }) => {
    return (
      <div className="marquee">
        <span>{text}</span>
      </div>
    );
  };
  return <div style={footerStyle}></div>;
}
export default ReservationCenterHeader;
above in is footer, do not change any naming conventions
