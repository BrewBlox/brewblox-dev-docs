# Troubleshooting

We're happy to help with any problems or questions that arise when using Brewblox. You can reach us on the forum: https://community.brewpi.com

Our first reply often consists of the same handful of questions. To save you some time, you may want to run through this checklist first.

- Is your system and firmware up-to-date? You can run `brewblox-ctl update` to check, or find the latest release date in https://brewblox.netlify.com/user/release_notes.html
- `brewblox-ctl log` generates all kinds of logs and diagnostic data for us to look at. It will also create a termbin.com link. You can copy that URL in your post.
- In the UI sidebar, bottom left corner there is a "bug" button. Click here, and choose "Export API errors to file". Please attach this file to your post.
- If your Spark service page is working, it may be useful to include your Blocks. You can export them by clicking on the "Actions" button in the top right corner of the service page, and then choosing "Import/Export Blocks". Please attach the generated file to your post.

## Known issues / workarounds

::: tip
We quickly fix most bugs we find. Some of them can't be fixed immediately, but have a temporary workaround.

We keep this list updated with the issues we're aware of, but haven't yet permanently resolved.
:::

**The Spark service keeps losing connection to the controller (wifi)**

The Wifi system library in the Spark controller has some serious issues. We've already fixed many of them, and are working on replacing the network library entirely.

Until then, be aware: any network reset will cause the Spark to lose Wifi connection. This includes router/AP resets, channel hops, and bad reception. We've also had reports of powerline adapters causing resets.

**After (re)starting, all my services and dashboards disappeared**

Sometimes the datastore service is very slow to get started. To check its status, you can view the Docker logs using the `docker-compose logs datastore` command.

After startup, the service is expected to print the following to its log:

```
datastore_1  | ****************************************************
datastore_1  | WARNING: CouchDB is running in Admin Party mode.
datastore_1  |          This will allow anyone with access to the
datastore_1  |          CouchDB port to access your database. In
datastore_1  |          Docker's default configuration, this is
datastore_1  |          effectively any other container on the same
datastore_1  |          system.
datastore_1  |          Use "-e COUCHDB_USER=admin -e COUCHDB_PASSWORD=password"
datastore_1  |          to set it in "docker run".
datastore_1  | ****************************************************
```

Depending on the host, this takes a few seconds to a few minutes.

You can also run `brewblox-ctl http wait https://localhost/datastore`. This will retry until it can connect to the datastore. When it reports success, refresh the UI.

## Frequently asked questions

**My service can't connect to my Spark over Wifi**

For reference, you can find the full guide on connection settings guide [here](./connect_settings.md).

Assuming you're still using the default settings (haven't added device-specific arguments to `docker-compose.yml`), you can use the following steps to troubleshoot your connection.

Is the Spark connected to Wifi?

- Did you run `brewblox-ctl wifi`?
- Does the Spark LCD show an IP address?

If the answer to either is no: run `brewblox-ctl wifi`.

Is the Spark accessible from your computer and the Raspberry Pi?

- Can you visit the Spark IP in your browser? It should show a short placeholder message.
- If you run `brewblox-ctl http get <SPARK_IP>`, do you see the html for the placeholder message?

If the answer to either is no, your Spark and Pi are likely using different subnets in your home network. Check your router configuration to allow them to communicate.

If the answer to all questions is yes, but the service still can't find your Spark, it may be a problem with mDNS.

By default, we use [multicast DNS](https://en.wikipedia.org/wiki/Multicast_DNS) to discover Sparks that are not connected over USB. In most - but not all - routers, mDNS is enabled by default. Check your router configuration for settings related to multicast DNS.

If you can't solve the problem in your router settings, it may be preferable to skip discovery, and add `--device-host=SPARK_IP` to your docker-compose.yml file. You can find the syntax in the [connection settings guide](./connect_settings.md).

When doing so, it is advised to assign a fixed IP address to the Spark in your router settings. (Also called "static DHCP lease").

**How do I display temperature in Fahrenheit?**

There are two settings: one for the UI and history, one for the Spark LCD.

UI:

- Go to the Spark service page
- In the top right corner, click on the Actions button (three vertical dots).
- Click on `Units`, and change the `Temperature` unit.

LCD:

- Go to the Spark service page.
- Click to select the `DisplaySettings` block in the block list.
- Click the widget toolbar button to switch to `Full` mode.
- Click on the current `Temperature Unit` to change the value.

**Why can removed or renamed blocks still be selected in the Graph Widget settings?**

History data is not changed when a block changes name. After a name change, the service simply starts publishing data under the new name.

The old name will disappear from the Graph widget settings 24 hours after the block is removed or renamed.

Units are part of the field name. For 24 hours after changing units from Celsius to Fahrenheit, you'll see both `spark-one/block/value[degC]` and `spark-one/block/value[degF]` in the Graph settings.
