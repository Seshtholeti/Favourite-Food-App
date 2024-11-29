const AWS = require('aws-sdk');
const S3 = new AWS.S3();
const connect = new AWS.Connect();
const csvParser = require('csv-parser');
const streamifier = require('streamifier');  // to handle buffer as a stream

// Your provided constants
const BUCKET_NAME = 'customeroutbound-data';
const FILE_NAME = 'CustomerOutboundNumber.csv';
const INSTANCE_ID = 'bd16d991-11c8-4d1e-9900-edd5ed4a9b21';
const CONTACT_FLOW_ID = '09f2c3c7-c424-4ad2-be1f-246be15b51a4';

exports.handler = async (event) => {
    try {
        // Step 1: Read the CSV file from S3
        const s3Params = {
            Bucket: BUCKET_NAME,
            Key: FILE_NAME
        };

        const s3Data = await S3.getObject(s3Params).promise();
        
        // Step 2: Parse the CSV content and extract phone numbers
        const phoneNumbers = await parseCSV(s3Data.Body);
        
        // Step 3: Query Amazon Connect for yesterday's call details
        const yesterday = new Date();
        yesterday.setDate(yesterday.getDate() - 1);
        const startTime = new Date(yesterday.setHours(0, 0, 0, 0)).toISOString();
        const endTime = new Date(yesterday.setHours(23, 59, 59, 999)).toISOString();

        // Query Amazon Connect for the contact details
        const contactDetails = await fetchContactDetails(startTime, endTime, phoneNumbers);

        // Step 4: Print the details of the calls made by those phone numbers
        console.log('Contact details:', contactDetails);
    } catch (error) {
        console.error('Error processing Lambda:', error);
        throw error;
    }
};

// Function to parse CSV file and extract phone numbers
const parseCSV = (csvData) => {
    return new Promise((resolve, reject) => {
        const phoneNumbers = [];
        const stream = streamifier.createReadStream(csvData);
        
        stream
            .pipe(csvParser({ separator: ';' }))  // Assuming CSV separator is semicolon
            .on('data', (row) => {
                // Extracting phone numbers from either 'PhoneNumber' or 'Name;PhoneNumber'
                const phoneNumber = row.PhoneNumber || row['Name;PhoneNumber']?.split(';')[1]?.trim();
                if (phoneNumber) {
                    let formattedNumber = phoneNumber.replace(/\D/g, '');  // Remove non-digit characters
                    if (formattedNumber.length === 10) {
                        formattedNumber = `+91${formattedNumber}`;  // Format as +91 for India
                    } else if (formattedNumber.length === 11) {
                        formattedNumber = `+1${formattedNumber}`;  // Format as +1 for US
                    }
                    phoneNumbers.push(formattedNumber);
                }
            })
            .on('end', () => resolve(phoneNumbers))
            .on('error', reject);
    });
};

// Function to fetch contact details from Amazon Connect
const fetchContactDetails = async (startTime, endTime, phoneNumbers) => {
    const contactDetails = [];

    // Iterate over phone numbers and fetch call details for each
    for (const phoneNumber of phoneNumbers) {
        const params = {
            InstanceId: INSTANCE_ID,
            StartTime: startTime,
            EndTime: endTime,
            Filters: {
                Channel: 'VOICE', // Assuming you want to fetch voice contacts
                Queue: 'your-queue-id', // Optionally filter by a specific queue
            }
        };

        try {
            const data = await connect.listContacts(params).promise();

            // Filter and process the records based on the phone number
            data.Contacts.forEach(contact => {
                if (contact.CustomerEndpoint && contact.CustomerEndpoint.Address === phoneNumber) {
                    const callDetails = {
                        PhoneNumber: phoneNumber,
                        ContactId: contact.ContactId,
                        AgentId: contact.AgentId,
                        Status: contact.ContactStatus,
                        Answered: contact.ContactStatus === 'COMPLETED' ? 'Yes' : 'No',
                        StartTime: contact.StartTime,
                        EndTime: contact.EndTime
                    };
                    contactDetails.push(callDetails);
                }
            });
        } catch (err) {
            console.error(`Error fetching details for ${phoneNumber}:`, err);
        }
    }

    return contactDetails;
};