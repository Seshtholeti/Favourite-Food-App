//     const phoneNumbers = [];
//     await new Promise((resolve, reject) => {
//       stream
//         .pipe(csvParser({ separator: ';' }))
//         .on('data', (row) => {
//           const phoneNumber = row.PhoneNumber || row['Name;PhoneNumber']?.split(';')[1]?.trim();
//           if (phoneNumber) {
//             let formattedNumber = phoneNumber.replace(/\D/g, '');
//             if (formattedNumber.length === 10) {
//               formattedNumber = `+91${formattedNumber}`;
//             } else if (formattedNumber.length === 11) {
//               formattedNumber = `+1${formattedNumber}`;
//             }
//             phoneNumbers.push(formattedNumber);
//           }
//         })
//         .on('end', resolve)
//         .on('error', reject);
//     });

//     if (phoneNumbers.length === 0) {
//       throw new Error('No phone numbers found in the CSV file.');
//     }
