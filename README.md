# Sign In Helper Project

This is an experimental job aid for tracking sign in to activities for groups that use ID cards that have barcodes.

## Using the project:

This solution is based on local HTML and Javascript. It does not rely on any server connections. You can run the code from Github pages at: [https://cloudassistprojects.github.io/sign-in/](https://hestsilas-29.github.io/CAP-checkin/) or you can copy/clone this repository to review the code or to run offline.

You can plug in a barcode scanner. It should be set to convey the barcode input and then provide a carriage return. This is normally the default for cheap USB scanners. If you get a 2-D barcode scanner, then you can scan either the front or the back of the CAP ID card. There is also a camera mode that reads barcodes from the webcam. This only works for the barcode on the back of the card, and is considerably slower than the dedicated scanner and can only handle 1D barcodes. (More Options, Camera Mode) Camera mode uses the front-facing ("selfie") camera, so on a tablet or phone the member holds their card up to the same side as the screen.

The data is cleared automatically each new day, based on the clock of the machine where it is being run, so it is important that the date, time, and timezone are set correctly.

Important - All data storage is local to the browser. If you do some sort of action that invalidates local storage then you will lose the current cache of data. I would recommend having a backup method for the first few times you use this to collect sign-in information.

Members record their attendance by scanning their ID card upon arrival. There is no capability for tracking sign-out.

At the end of the meeting, or once all members have signed in, you can click the "More Options" and then export the data using the "File", "Clipboard", or "Both" options. Until you have a good system for entering data, I recommend "both" so that the file generated locally on the machine can be used as a backup.

- Clipboard - copies all CAPIDs separated by commas into the system clipboard so they can be pasted into your Attendance Log.
- File - will name a file for the current date and trigger the browser to download of a file with the CAPIDs, comma separated
- Both - does both of the above at the same time.

## Advanced options

### Webhook function

If a value is configured in the Webhook setting, any time you export data (clipboard/file/both) the browser will also POST a json object to the specified endpoint. I have used a PowerAutomate flow, and an Azure Logic App that receives the POST and sends an email to our commanders and admin staff so that someone can enter attendance without having to log in to eServices on the attendance laptop. Here's an example of what the webhook post sends:

    {
            "meeting_date": "7/18/2023",
            "attendees": [
                "111111",
                "222222"
            ]
    }

### Autosend function

If a value is configured in this setting, the system will automatically trigger the webhook export at the configured time. This must be entered using 24-hour time like: 21:15 for 9:15PM.

### Recovery of unsent records

The system keeps a count of records that have not yet been successfully sent to the webhook. Each sign-in increases that count, and it is only cleared when a webhook POST comes back with a success (HTTP 2xx) response or when the data is cleared. If the POST fails - the network is down, the endpoint is unreachable, or the server returns an error - the records stay marked as pending, and details of the failure are written to the browser console.

If records are still pending when the date rolls over, the next scan on the new day will automatically POST the previous day's data to the webhook before clearing it. The recovered post is labeled with the earlier date so that it is easy to spot on the receiving end:

    {
            "meeting_date": "7/18/2023 (RECOVERED)",
            "attendees": [
                "111111",
                "222222"
            ]
    }

This means a meeting where someone forgot to export, or where the webhook failed, can still get its attendance out, but only if the browser is opened again on the same machine and a sign-in is recorded. It is not a substitute for exporting at the end of the meeting, and it does not apply if no webhook is configured.
