import React, { useState } from "react";
import axios from "axios";

const MainComponent = () => {
  const [searchQuery, setSearchQuery] = useState(
    "bd16d991-11c8-4d1e-9900-edd5ed4a9b21"
  );
  const [components, setComponents] = useState([]);
  const [selectedComponents, setSelectedComponents] = useState([]);
  const [responseData, setResponseData] = useState(null);
  const [availableTags, setAvailableTags] = useState([]);
  const [selectedTag, setSelectedTag] = useState(null);
  const [showSpinnerModal, setShowSpinnerModal] = useState(false);
  const [newTagKey, setNewTagKey] = useState("");
  const [newTagValue, setNewTagValue] = useState("");

  const handleSearchInputChange = (event) => {
    setSearchQuery(event.target.value);
  };

  const handleFetchClick = () => {
    setShowSpinnerModal(true);
    const apiComponents =
      "https://guixfoyppb.execute-api.us-east-1.amazonaws.com/tagging/";
    axios
      .post(apiComponents, { intent: "agents" })
      .then((response) => {
        setComponents(Object.keys(response.data));
        setResponseData(response.data);
        setShowSpinnerModal(false);
      })
      .catch((error) => {
        console.error("Error fetching components:", error);
        setShowSpinnerModal(false);
      });
  };

  const fetchTagsForARN = (arn) => {
    const api = "https://l78y00q47e.execute-api.us-east-1.amazonaws.com/test/";
    axios
      .post(api, { arn: arn, intent: "fetch" })
      .then((response) => {
        console.log(response.data);
        const tagsArray = Object.entries(response.data);
        setAvailableTags(tagsArray);
      })
      .catch((error) => {
        console.error("Error fetching tags:", error);
      });
  };

  const handleComponentSelect = (component) => {
    if (selectedComponents.includes(component)) {
      setSelectedComponents(selectedComponents.filter((c) => c !== component));
    } else {
      setSelectedComponents([...selectedComponents, component]);
    }
  };

  const handleTagSelect = (tag) => {
    setSelectedTag(tag);
  };

  const handleRemoveTag = (arn, tagName) => {
    const tagArr = [tagName];
    const removeTagApi =
      "https://l78y00q47e.execute-api.us-east-1.amazonaws.com/test/";
    axios
      .post(removeTagApi, { arn: arn, tagName: tagArr, intent: "unTag" })
      .then((response) => {
        setAvailableTags(
          availableTags.filter(([key, value]) => key !== tagName)
        );
      })
      .catch((error) => {
        console.error("Error removing tag:", error);
      });
  };

  const handleAddTag = () => {
    setShowSpinnerModal(true);
    const addTagApi =
      "https://l78y00q47e.execute-api.us-east-1.amazonaws.com/test/";
    axios
      .post(addTagApi, {
        arn: selectedComponents,
        tagName: newTagKey,
        tagValue: newTagValue,
        intent: "addTag",
      })
      .then((response) => {
        fetchTagsForARN(selectedComponents[0]); // Refresh tags after adding
        setNewTagKey("");
        setNewTagValue("");
        setShowSpinnerModal(false);
      })
      .catch((error) => {
        console.error("Error adding tag:", error);
        setShowSpinnerModal(false);
      });
  };

  return (
    <div>
      <div style={{ marginBottom: "20px" }}>
        <h2 style={{ marginBottom: "20px" }}>Search Component</h2>
        <div style={{ display: "flex", alignItems: "center" }}>
          <select
            value={searchQuery}
            onChange={handleSearchInputChange}
            style={{
              width: "400px",
              marginRight: "10px",
              marginLeft: "400px",
              padding: "8px",
              border: "1px solid #ccc",
              borderRadius: "5px",
              fontSize: "16px",
            }}
          >
            <option value="bd16d991-11c8-4d1e-9900-edd5ed4a9b21">
              bd16d991-11c8-4d1e-9900-edd5ed4a9b21
            </option>
          </select>
          <button
            onClick={handleFetchClick}
            style={{
              padding: "10px 20px",
              backgroundColor: "#007bff",
              color: "#fff",
              border: "none",
              borderRadius: "5px",
              cursor: "pointer",
              fontSize: "16px",
            }}
          >
            Fetch
          </button>
        </div>
      </div>
      {showSpinnerModal && (
        <
div
          style={{
            position: "fixed",
            top: 0,
            left: 0,
            width: "100%",
            height: "100%",
            backgroundColor: "rgba(0, 0, 0, 0.5)",
            zIndex: 9999,
            display: "flex",
            justifyContent: "center",
            alignItems: "center",
          }}
        >
          <div
            style={{
              width: "100px",
              height: "100px",
              backgroundColor: "#fff",
              borderRadius: "10px",
              display: "flex",
              justifyContent: "center",
              alignItems: "center",
              boxShadow: "0 0 10px rgba(0, 0, 0, 0.3)",
            }}
          >
            <div
              style={{
                border: "4px solid rgba(0, 0, 0, 0.1)",
                borderTop: "4px solid #007bff",
                borderRadius: "50%",
                width: "40px",
                height: "40px",
                animation: "spin 1s linear infinite",
              }}
            ></div>
          </div>
        </div>
      )}
      <div style={{ display: "flex", padding: "20px" }}>
        <div
          style={{
            flex: "1",
            marginRight: "20px",
            backgroundColor: "#D3D3D3",
            borderRadius: "10px",
            boxShadow: "0 0 10px rgba(0, 0, 0, 0.1)",
            padding: "20px",
          }}
        >
          <h2 style={{ marginBottom: "20px", fontSize: "20px" }}>Components</h2>
          {components.map((component, index) => (
            <div key={index}>
              <input
                type="checkbox"
                id={component}
                name={component}
                checked={selectedComponents.includes(component)}
                onChange={() => handleComponentSelect(component)}
              />
              <label htmlFor={component}>{component}</label>
            </div>
          ))}
        </div>
        <div
          style={{
            flex: "1",
            backgroundColor: "#D3D3D3",
            borderRadius: "10px",
            boxShadow: "0 0 10px rgba(0, 0, 0, 0.1)",
            padding: "20px",
          }}
        >
          <h2 style={{ marginBottom: "20px", fontSize: "20px" }}>Add Tag</h2>
          <input
            type="text"
            placeholder="Key"
            value={newTagKey}
            onChange={(e) => setNewTagKey(e.target.value)}
            style={{ marginBottom: "10px", width: "80%", padding: "8px" }}
          />
          <input
            type="text"
            placeholder="Value"
            value={newTagValue}
            onChange={(e) => setNewTagValue(e.target.value)}
            style={{ marginBottom: "10px", width: "80%", padding: "8px" }}
          />
          <button
            onClick={handleAddTag}
            style={{
              padding: "10px 20px",
              marginLeft: "270px",
              backgroundColor: "#28a745",
              color: "#fff",
              border: "none",
              borderRadius: "5px",
              cursor: "pointer",
              fontSize: "16px",
              display: "flex",
            }}
          >
            Add
          </button>
        </div>
      </div>
      <div style={{ display: "flex", padding: "20px" }}>
        <div
          style={{
            flex: "1",
            backgroundColor: "#D3D3D3",
            borderRadius: "10px",
            boxShadow: "0 0 10px rgba(0, 0, 0, 0.1)",
            padding: "20px",
          }}
        >
          <h2 style={{ marginBottom: "20px", fontSize: "20px" }}>Available Tags</h2>
          <select
            value={selectedTag}
            onChange={(event) => handleTagSelect(event.target.value)}
            style={{
              width: "100%",
              padding: "8px",
              border: "1px solid #ccc",
              borderRadius: "5px",
              fontSize: "16px",
            }}
          >
            <option value="">Select a tag</option>
            {availableTags.map(([key, value], index) => (
              <option key={index} value={key}>
                {key}: {value}
              </option>
            ))}
          </select>
          {selectedTag && (
            <button
              onClick={() => handleRemoveTag(selectedComponents[0], selectedTag)}
              style={{
                padding: "10px 20px",
                backgroundColor: "#dc3545",
                color: "#fff",
                border: "none",
                borderRadius: "5px",
                cursor: "pointer",
                fontSize: "16px",
                marginLeft: "10px",
                marginTop: "40px",
              }}
            >
              Remove
            </button>
          )}
        </div>
      </div>
    </div>
  );
};

export default MainComponent;