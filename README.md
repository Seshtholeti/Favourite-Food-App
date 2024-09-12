.header {
  background-color: #1890ff;
  padding: 20px;
  text-align: center;
}
.header-content {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
}
.ant-table {
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin-top: 20px;
  font-size: 16px; /* Increased font size for better readability */
}
.ant-table th {
  background-color: #1890ff; /* Header color */
  color: white;
  font-size: 18px; /* Increased font size */
  font-weight: bold;
  padding: 12px;
}
.ant-table td {
  padding: 12px;
}
.gold-row {
  background-color: #fffde7; /* Light yellow for gold */
}
.silver-row {
  background-color: #f3f6f9; /* Light gray for silver */
}
.bronze-row {
  background-color: #f9f9f9; /* Light background for bronze */
}
.ant-badge {
  font-weight: bold;
}
.top-performers {
  margin-bottom: 20px;
}
.top-performers-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}
.top-performer-card {
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  padding: 20px;
  text-align: center;
  margin: 10px;
  width: 220px;
  transition: transform 0.2s;
}
.top-performer-card:hover {
  transform: scale(1.05);
}
.badge-container {
  display: flex;
  align-items: center;
  justify-content: center;
}
.gold-badge {
  color: #ffd700;
  font-size: 24px;
  margin-right: 8px;
}
.performance-score {
  display: block;
  margin: 10px 0;
}
.car-progress-container {
  position: relative;
  width: 100%;
  height: 50px;
  margin: 10px 0;
}
.track {
  background-color: #e0e0e0;
  border-radius: 15px;
  overflow: hidden;
  height: 100%;
  position: relative;
}
.progress-bar {
  background-color: #1890ff;
  height: 100%;
  border-radius: 15px;
  transition: width 0.5s ease-in-out; /* Smooth transition */
}
.progress-text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-weight: bold;
}
.top-performer-container {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}
.sparkle {
  position: absolute;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  animation: sparkle 1.5s infinite;
}
@keyframes sparkle {
  0% {
    opacity: 0;
    transform: scale(0.5);
  }
  50% {
    opacity: 1;
    transform: scale(1.2);
  }
  100% {
    opacity: 0;
    transform: scale(1);
  }
}
.sparkle:nth-child(1) {
  background-color: #ff0000; /* Red */
  left: 10%;
  top: 10%;
  animation-delay: 0s;
}
.sparkle:nth-child(2) {
  background-color: #00ff00; /* Green */
  left: 30%;
  top: 20%;
  animation-delay: 0.3s;
}
.sparkle:nth-child(3) {
  background-color: #0000ff; /* Blue */
  left: 50%;
  top: 10%;
  animation-delay: 0.6s;
}
.sparkle:nth-child(4) {
  background-color: #ffff00; /* Yellow */
  left: 70%;
  top: 20%;
  animation-delay: 0.9s;
}
.sparkle:nth-child(5) {
  background-color: #ff00ff; /* Magenta */
  left: 90%;
  top: 10%;
  animation-delay: 1.2s;
}
