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
    height: "70px",
    display: "flex",
    alignItems: "center",
    fontSize: "50px",
    width: "98vw",
    justifyContent: "space-between",
    position: "relative",
    bottom: "5px",
  };
  const headerTitle = {
    paddingBottom: "5px",
    // width: "100%",
    marginLeft: "60px",
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
    marginRight: "60px",
  };
  const iconStyle = {
    marginLeft: "10px",
    fontSize: "30px",
    color: "white",
    cursor: "pointer",
  };
  const iconHoverStyle = {
    color: "#c9302c",
  };
  const logoutTextStyle = {
    fontSize: "10px",
    marginLeft: "15px",
    visibility: isHovered ? "visible" : "hidden",
  };
  return (
    <div style={headerStyle}>
      <div style={headerTitle}>Anrufstatistik</div>
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
