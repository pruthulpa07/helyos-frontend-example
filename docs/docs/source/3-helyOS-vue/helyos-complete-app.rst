Complete helyOS App
===================
This chapter will demonstrates a whole **helyOS-based** web app built by **Vue.js** with ``Pinia`` stores and **Service Layer** introduced in previous chapters.

Static Resources
----------------

Firstly, add some static resources into *assets* folder:

*./assets/html/helyos-demo.html*

.. code:: 

    <header>
        <div id="title">Hello, helyOS!</div>
    </header>

    <section>
        <nav>
            <ul>
                <li>
                    <div id="yard-menu">
                        Yard:
                        <select id="selected-yard" v-model="yardStore.selectedYard" @change="initYard"
                            @mouseenter="hintMsg = 'helyosService.yard.list(condition?: Partial<H_Yard>): Promise<H_Yard[]>'"
                            @mouseleave="hintMsg = 'HelyOS'">
                            <option v-for="yard in yardStore.yards" :value="yard.id">{{ yard.name }}</option>
                        </select>
                        {{ yardStore.selectedYard }}
                    </div>
                </li>
                <li>
                    Upload a GeoJson file:
                    <div id="geojson-bar">
                        <input type="file" id="upload-geojson-btn" class="custom-input" @change="uploadGeoJson">
                        <label for="upload-geojson-btn" class="custom-btn"
                            @mouseenter="hintMsg = 'L.geoJSON(<Object> geojson?, <GeoJSON options> options?).addTo(map)'"
                            @mouseleave="hintMsg = 'HelyOS'">Choose a file</label>
                        <!-- </div>
                        <div> -->
                        <button class="custom-btn" id="push-geojson-btn" @click="pushGeoJson"
                            @mouseenter="hintMsg = 'helyosService.shapes.create(shape: Partial<H_Shape>): Promise<any>'"
                            @mouseleave="hintMsg = 'HelyOS'">Push</button>
                        <button class="custom-btn" id="delete-geojson-btn" @click="deleteGeoJson"
                            @mouseenter="hintMsg = 'helyosService.shapes.delete(shapeId: any): Promise<any>'"
                            @mouseleave="hintMsg = 'HelyOS'">Delete</button>
                    </div>
                </li>
                <li>
                    <div id="icon-bar">
                        Upload a truck icon (.PNG):
                        <label>
                            <input type="file" id="upload-icon-btn" class="custom-input" @change="updateToolIcon">
                            <label for="upload-icon-btn" class="custom-btn"
                                @mouseenter="hintMsg = 'helyosService.agents.patch(tool: Partial<H_Agents>): Promise<any>'"
                                @mouseleave="hintMsg = 'HelyOS'">Choose a file</label>
                        </label>
                    </div>
                </li>
                <li id="tool-status-bar">
                    <div class="tool-status">Tool Status: {{ }}</div>
                    <div class="tool-status"
                        @mouseenter="hintMsg = 'helyosService.socket.on(\'new_agent_poses\', (updates: any) => { console.log(updates); }'"
                        @mouseleave="hintMsg = 'HelyOS'">
                        {{ toolStore.selectedToolInfo }}
                    </div>
                </li>
                <li id="mission-bar">
                    <div class="mission-info">
                        Mission:
                        <select v-model="workProcessStore.selectedMission" id="selected-mission" @change="initMission"
                            @mouseenter="hintMsg = 'helyosService.workProcessType.list(condition: Partial<H_WorkProcessType>): Promise<any>'"
                            @mouseleave="hintMsg = 'HelyOS'">
                            <option v-for="wpType in workProcessStore.workProcessType" :value="wpType.name">
                                {{ wpType.name }}
                            </option>
                            <option></option>
                        </select>
                        {{ workProcessStore.selectedMission }}
                    </div>
                    <div class="mission-info">
                        <div>Request: <br>{{ requestMsg }}</div>
                        <textarea v-model="requestMsg" class="mission-info-textarea" contenteditable> </textarea>
                    </div>
                    <div class="mission-info">
                        <div>Setting: <br>{{ settingMsg }}</div>
                        <textarea v-model="settingMsg" class="mission-info-textarea"> </textarea>
                    </div>
                    <div class="mission-info">
                        <button @click="createMission" class="custom-btn"
                            @mouseenter="hintMsg = 'helyosService.workProcess.create(workProcess: Partial<H_WorkProcess>): Promise<any>'"
                            @mouseleave="hintMsg = 'HelyOS'">Dispatch</button>
                    </div>
                </li>
                <li id="hint">
                    Hints:
                    <div>{{ hintMsg }}</div>
                </li>
            </ul>
        </nav>

        <div id="map-interface">
            <Map ref="mapRef"></Map>
        </div>
    </section>

    <footer>
        <p>Footer</p>
    </footer>

*./assets/css/helyos-demo.css*

.. code:: css

    /* Header */
    header {
        background-color: #666;
        margin-top: 0;
        text-align: center;
        color: white;
    }

    #title {
        padding: 15px;
        margin-top: 0;
        font-size: 30px;
        font-weight: bold;
    }

    /* Flexboxes container */
    section {
        /* display: -webkit-flex; */
        display: flex;
    }

    /* Navigation menu */
    nav {
        width: 20%;
        height: 100%;
        /* -webkit-flex: 1;
        -ms-flex: 1;
        flex: 1; */
        background: rgba(204, 204, 204, 0);
        padding: 5px;
    }

    /* Navigation menu list */
    nav ul {
        list-style-type: none;
        padding: 0;
        margin: 0;
    }

    nav ul li {
        margin: 10px 0 0 0;
        padding: 10px;
        color: white;
        background-color: rgb(86, 143, 121);

    }

    /* Select box */
    select {
        width: auto;
        margin: 5px 0 5px 0;
    }

    /* Buttons */
    .custom-btn {
        font-size: 16px;
        font-weight: 500;
        padding: 5px;
        margin: 5px 5px 0 0;
        border: 0;
        border-radius: 5px;
        color: white;
        background-color: rgb(51, 92, 78);
        display: inline-block;
        cursor: pointer;
    }


    .custom-btn:hover {
        background-color: rgb(245, 86, 12);
    }

    .custom-input {
        width: 0.1px;
        height: 0.1px;
        opacity: 0;
        overflow: hidden;
        position: absolute;
        z-index: -1;
    }

    /* Tool status information panel */
    .tool-status {
        margin: 5px;
        line-height: 1.5;
    }


    /* Mission dialogue */
    .mission-info {
        word-wrap: break-word;
        margin: 5px;
    }

    .mission-info-textarea {
        margin-top: 5px;
        height: 60px;
        width: 100%;
    }

    /* Hint */
    .hint {
        display: block;
    }

    /* Right volume */
    #map-interface {
        /* -webkit-flex: 3;
        -ms-flex: 3;
        flex: 3; */
        width: 80%;
        height: 90vh;
        /* background-color: rgb(63, 64, 65); */
        padding: 0px;
    }

    /* Footer */
    footer {
        background-color: #777;
        padding: 5px;
        text-align: center;
        color: white;
    }

helyOS Interface
----------------

To initialize the connection and data fetching, you have to improve *Login.vue* a bit, which was built in :ref:`Web Interface and Routing<WebInterfaceAndRouting>` before.

Make sure to initialize helyOS connection and data fetching after login token confirmed.

*./components/Login.vue*

.. code::

    <template>
        <div class="login-form">
            <h1>Welcome to helyOS hello-world application</h1>
            <div>Username:
                <input type="text" v-model="loginForm.username" placeholder="user name" />
            </div>
            <div>Password:
                <input type="password" v-model="loginForm.password" placeholder="password" />
            </div>
            <button @click="login">Login</button>
        </div>
    </template>

    <script setup lang="ts">
    import { ref } from 'vue'
    import { useUserStore, type User } from '@/stores/user-store';
    import { useRouter } from 'vue-router'
    import * as HS from '@/services/helyos-service';

    // define loginForm
    const loginForm = ref({
        username: '',
        password: '',
        token: ''
    } as User);

    // define userStore
    const userStore = useUserStore();
    // define router
    const router = useRouter();

    // login
    const login = async () => {

        // login into helyOS and get the helyOS Token
        const helyosToken = await HS.helyosLogin(loginForm.value.username, loginForm.value.password);
        console.log("tk", helyosToken);

        // login in successfully
        if (helyosToken) {
            loginForm.value.token = helyosToken.jwtToken;
            // store user information
            userStore.setUser(loginForm.value);

            // initialize helyOS connection and data fetching
            const connected = await HS.helyosConnect();
            console.log(connected);

            // routing to next page
            router.push({
                name: "demo",
            })
        } else {
            alert("Incorrect username or password!")
        }
    }

    defineExpose({
        login,
    })

    </script>

    <style scoped>
    .login-form {
        background-color: lightgray;
        margin: auto;
        width: 50%;
        border: 3px solid green;
        padding: 10px;
        text-align: center;
        /* padding: 20%; */
    }

    .login-form input {
        margin: 10px;
    }

    .login-form button {
        margin: 10px;
    }
    </style>



Then, the browser will route to another component *Helyos.vue*, which is the main helyOS interface:

*./components/Helyos.vue*

.. code:: html

    <template src="./../assets/html/helyos-demo.html"></template>  
    <style src="./../assets/css/helyos-demo.css"></style>

    <script setup lang="ts">
    import { onMounted, ref, watch } from 'vue'
    import Map from "./LeafletMap.vue";
    import * as HS from '@/services/helyos-service';
    import { useYardStore } from '@/stores/yard-store';
    import { useUserStore } from '@/stores/user-store';
    import { useMapObjectStore } from '@/stores/map-object-store';
    import { useToolStore } from '@/stores/tool-store';
    import { useWorkProcessStore } from '@/stores/work-process-store';
    import { useLeafletMapStore } from '@/stores/leaflet-map-store';

    const mapStore = useLeafletMapStore(); // leaflet map store
    const mapRef = ref(); // reactive variable of leaflet map

    const userStore = useUserStore(); // helyos user store
    const yardStore = useYardStore(); // helyos yards Store
    const mapObjectStore = useMapObjectStore();// helyos map object store
    const toolStore = useToolStore(); // helyos tools store
    const workProcessStore = useWorkProcessStore(); // helyos work process store

    const hintMsg = ref("Welcome to HelyOS");

    // initialize yard
    const initYard = () => {
        console.log("Initiate yard", yardStore.selectedYard);
        // set new map view of current yard
        const currentYard = yardStore.getCurrentYard();
        const originLatLon = {
            lat: currentYard[0].lat,
            lon: currentYard[0].lon
        }
        mapRef.value.updateMap(originLatLon.lat, originLatLon.lon);

        // display MapObjects of current yard from MapObject store
        initMapObjects();

        // display tools of current yard from tool store
        initTools();
    }

    // initialize tools layer
    const initTools = () => {
        console.log(yardStore.selectedYard);

        const tools = toolStore.filterToolByYard(yardStore.selectedYard);
        console.log("tools in current yard", tools);
        tools.forEach((tool) => {
            mapRef.value.toolMarker(tool);
        })
    }

    // watch tool status
    const unwatch = watch(
        () => toolStore.ifSubscription,
        (ifSubscription) => {
            // console.log("watching", ifsubscription);
            if (ifSubscription && yardStore.selectedYard) {

                toolStore.tools.forEach((tool) => {

                    if ('marker' in tool) {
                        const toolPose = {
                            lat: tool.y,
                            lng: tool.x
                        }
                        mapRef.value.updateMarkerLatLng(tool, toolPose);
                    }

                    if (toolStore.selectedTool) {
                        toolStore.updateSelectedTool();
                    }
                })
            }
            toolStore.ifSubscription = 0;
        },
        { deep: true }
    )

    // upload a image file as truck icon
    const updateToolIcon = (ev: any) => {
        console.log("upload icon");

        // read png file
        const PNGFile = ev.target.files[0];

        // check if file is a .geojson or .json format
        const checkPNGFile = PNGFile.name.split(".").pop();
        if (checkPNGFile === "png" || checkPNGFile === "PNG") {
            // read json file as text
            const reader = new FileReader();
            reader.readAsDataURL(PNGFile);
            reader.onload = (file) => {
                // console.log(tool);
                const iconURL = file.target?.result as string;
                toolStore.patchToolIcon(iconURL);
                initYard();
            }
        } else {
            alert("Wrong file format! Please upload png format.")
        }
    }

    // initialize MapObjects layer
    const initMapObjects = () => {
        // get mapObjects from MapObject store
        const mapObjects = mapObjectStore.filterMapObjectByYard(yardStore.selectedYard);
        // if (yardStore.selectedYard === "4") { // geojson format MapObjects
        console.log("MapObjects in current yard", mapObjects);

        mapObjects.forEach((mapObject) => {
            mapRef.value.geoJsonDisplay(mapObject.data);
        })

        // clear temporary geojson object
        geoJsonObj.value = undefined;
    }

    // upload GeoJSON file and display objects on the map
    const geoJsonObj = ref(); // geojson object container
    const uploadGeoJson = (ev: any) => {
        console.log("upload geojson");

        // read json file
        const jsonFile = ev.target.files[0];

        // check if file is a .geojson or .json format
        const checkJsonFile = jsonFile.name.split(".").pop();
        if (checkJsonFile === "json" || checkJsonFile === "geojson") {

            // read json file as text
            const reader = new FileReader();
            reader.readAsText(jsonFile);
            reader.onload = (file) => {
                // convert text to json object
                geoJsonObj.value = JSON.parse(file.target?.result as any); // FeatureCollection from geojson/json file 
                // display geojson object
                mapRef.value.geoJsonDisplay(geoJsonObj.value);
            }

        } else {
            alert("Wrong file format!")
        }
    }

    // push uploaded GeoJSON data to helyos database
    const pushGeoJson = async () => {
        if (geoJsonObj.value) {
            const newMapObject = {
                yardId: yardStore.selectedYard,
                // isObstacle: true,
                type: "obstacle",
                dataFormat: "GeoJSON",
                name: "obstacle",
                data: geoJsonObj.value
            }
            console.log(newMapObject);
            mapObjectStore.pushMapObject(newMapObject);
        } else {
            alert("Push failed: please upload geojson file firstly.");
        }
    }

    // delete all MapObjects of current yard
    const deleteGeoJson = () => {
        mapObjectStore.deleteMapObjectsByYard(yardStore.selectedYard);
        initYard();
    }

    // create a new mission dispatch it via workProcess store
    const settingMsg = ref("{}");
    const requestMsg = ref("{}");
    const initMission = () => {
        console.log("mission", workProcessStore.selectedMission); // workProcessType.name

        if (workProcessStore.selectedMission) { // if mission selected
            if (!toolStore.selectedTool) { // if tool not selected
                requestMsg.value = "{\"results\": [{\"agent_id\": , \"assignment\": { \"destination\": { \"x\": , \"y\": , \"orientations\":[0,0] }}}]}";
            } else if (!mapStore.onClickCoords) { // if map not clicked
                requestMsg.value = "{\"results\": [{\"agent_id\": " + toolStore.selectedTool.id + ", \"assignment\": { \"destination\": { \"x\": , \"y\": , \"orientations\":[0,0] }}}]}";
            } else { // if tool, mission are selected and map clicked
                requestMsg.value = "{\"results\": [{\"agent_id\": " + toolStore.selectedTool.id + ", \"assignment\": { \"destination\": { \"x\":" + mapStore.onClickCoords.lng + ", \"y\":" + mapStore.onClickCoords.lat + ", \"orientations\":[0,0] }}}]}";
            }
        } else {

            requestMsg.value = "{}";
        }
    }

    // watch map onClick latlng coordinates
    const unwatchOnClickMap = watch(
        () => mapStore.onClickCoords,
        (coords) => {
            if (!toolStore.selectedTool) {
                alert("Please select a tool firstly!")
            } else if (workProcessStore.selectedMission) {
                requestMsg.value = "{\"results\": [{\"agent_id\": " + toolStore.selectedTool.id + ", \"assignment\": { \"destination\": { \"x\":" + coords.lng + ", \"y\":" + coords.lat + ", \"orientations\":[0,0] }}}]}";
            }
            // console.log("coords", coords);
        }
    )

    // dispatch the mission
    const createMission = () => {
        console.log("mission", workProcessStore.selectedMission); // workProcessType.name
        console.log("request", requestMsg.value);
        console.log("setting", settingMsg.value);

        if (!toolStore.selectedTool) {
            alert("Please select a tool firstly!")
        } else if (workProcessStore.selectedMission == "") {
            alert("Please select a mission!")
        } else if (requestMsg.value === "{}") {
            alert("Request must not be empty!")
        } else {
            workProcessStore.dispatchMission(Number(toolStore.selectedTool.id), yardStore.selectedYard, requestMsg.value, settingMsg.value);
        }

    }


    onMounted(() => {
        setTimeout(() => {
            initYard();
        }, 1000)
    })

    </script>



helyOS Hello World
------------------

Congratulations! This is the end of this tutorial. If you follow this whole tutorial and everything works well, you will get a helyOS-based web app like the following 
figure shown.

.. figure:: ./img/helyos_demo.png
    :align: center
    :width: 500pt

    helyOS hello world app

The complete hello helyOS project can be accessed by `helyOS frontend example <https://github.com/FraunhoferIVI/helyos-frontend-example>`_.