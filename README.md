```css
.header {
  background-color: #00008b; /* Dark blue for the header */
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
  font-size: 16px; /* Base font size for the table */
  border-radius: 8px; /* Rounded corners for the table */
  overflow: hidden; /* Ensures rounded corners are visible */
}

.ant-table th {
  background-color: black; /* Black background for table headers */
  color: white; /* White text color for headers */
  font-size: 18px; /* Font size for header text */
  font-weight: bold; /* Bold text for headers */
  padding: 12px; /* Padding for header cells */
}

.ant-table td {
  padding: 12px; /* Padding for table cells */
  font-size: 16px; /* Font size for table cell text */
  background-color: #f6f6f6; /* Light gray background for cells */
}

.gold-row {
  background-color: #fffde7; /* Light yellow for gold performance rows */
}

.silver-row {
  background-color: #f3f6f9; /* Light gray for silver performance rows */
}

.bronze-row {
  background-color: #f9f9f9; /* Light background for bronze performance rows */
}

.ant-badge {
  font-weight: bold; /* Bold text for badges */
}

.ant-table-expanded-row {
  background-color: #f6f6f6; /* Background color for expanded rows */
  padding: 16px; /* Padding for expanded row */
}

.ant-table-expanded-row > td {
  border: none; /* Remove border for expanded row */
}

.expanded-details {
  margin: 12px 0; /* Margin for spacing between details */
}

.expanded-label {
  font-weight: bold; /* Bold for labels in expanded details */
  font-size: 18px; /* Larger font size for labels */
}

.expanded-value {
  font-size: 16px; /* Regular font size for values */
  margin-left: 8px; /* Space between label and value */
}

```

```jsx
import React, { useEffect, useState } from "react";
import { Table, Layout, Typography, Badge } from "antd";
import axios from "axios";
import "./GamificationUI.css";

const { Header, Content } = Layout;
const { Title, Text } = Typography;

const GamificationUI = () => {
  const [agents, setAgents] = useState([]);
  const [expandedRowKeys, setExpandedRowKeys] = useState([]);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await axios.get(
          "https://rrjboaljfmf5vyhuienk5mzszi0weebt.lambda-url.us-east-1.on.aws/"
        );
        setAgents(response.data);
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    };
    fetchData();
  }, []);

  const columns = [
    {
      title: "Agent Name",
      dataIndex: "agent_name",
      key: "agent_name",
      width: "20%",
    },
    {
      title: "Performance Score",
      dataIndex: "performance_score",
      key: "performance_score",
      width: "15%",
    },
    {
      title: "Agent Talk Time (s)",
      dataIndex: "agent_talk_time",
      key: "agent_talk_time",
      width: "15%",
    },
    {
      title: "Agent Calls Count",
      dataIndex: "agent_calls_count",
      key: "agent_calls_count",
      width: "15%",
    },
    {
      title: "Customer Sentiments Score",
      dataIndex: "customer_sentiments_score",
      key: "customer_sentiments_score",
      width: "15%",
    },
    {
      title: "Badge",
      dataIndex: "performance_score",
      key: "badge",
      width: "20%",
      render: (score) => (
        <Badge
          count={
            <img
              src={
                score > 400
                  ? "gold_badge.png" // Replace with actual path to gold badge image
                  : score > 300
                  ? "silver_badge.png" // Replace with actual path to silver badge image
                  : "bronze_badge.png" // Replace with actual path to bronze badge image
              }
              alt="Badge"
              style={{ width: "50px" }}
            />
          }
        />
      ),
    },
  ];

  const expandedRowRender = (record) => (
    <div style={{ padding: "16px" }}>
      <div style={{ marginBottom: "12px" }}>
        <Text strong style={{ fontSize: "18px" }}>
          Agent Sentiments Score:
        </Text>
        <Text style={{ fontSize: "16px", marginLeft: "8px" }}>
          {record.agent_sentiments_score}
        </Text>
      </div>
      <div style={{ marginBottom: "12px" }}>
        <Text strong style={{ fontSize: "18px" }}>
          Agent Non-Talk Time (s):
        </Text>
        <Text style={{ fontSize: "16px", marginLeft: "8px" }}>
          {record.agent_non_talk_time}
        </Text>
      </div>
      <div style={{ marginBottom: "12px" }}>
        <Text strong style={{ fontSize: "18px" }}>
          Agent ID:
        </Text>
        <Text style={{ fontSize: "16px", marginLeft: "8px" }}>
          {record.agent_id}
        </Text>
      </div>
      <div style={{ marginBottom: "12px" }}>
        <Text strong style={{ fontSize: "18px" }}>
          Customer Sentiments Score:
        </Text>
        <Text style={{ fontSize: "16px", marginLeft: "8px" }}>
          {record.customer_sentiments_score}
        </Text>
      </div>
    </div>
  );

  const onExpand = (expanded, record) => {
    if (expanded) {
      setExpandedRowKeys([record.agent_id]);
    } else {
      setExpandedRowKeys([]);
    }
  };

  return (
    <Layout style={{ minHeight: "100vh" }}>
      <Header className="header">
        <div className="header-content">
          <Title style={{ color: "white" }} level={2}>
            Agent Performance Leaderboard
          </Title>
        </div>
      </Header>
      <Content style={{ padding: "20px" }}>
        <Table
          dataSource={agents}
          columns={columns}
          rowKey="agent_id"
          pagination={false}
          expandedRowKeys={expandedRowKeys}
          onExpand={onExpand}
          expandedRowRender={expandedRowRender}
          rowClassName={(record) => {
            if (record.performance_score > 400) {
              return "gold-row";
            } else if (record.performance_score > 300) {
              return "silver-row";
            } else {
              return "bronze-row";
            }
          }}
          style={{
            backgroundColor: "#fff",
            borderRadius: "8px",
            boxShadow: "0 2px 8px rgba(0, 0, 0, 0.1)",
            marginTop: "20px",
            fontSize: "16px",
          }}
        />
      </Content>
    </Layout>
  );
};

export default GamificationUI;

```
