<template>

  <HCGenericDialog
    @confirm="updateGui"
    ref="updateGuiDialog"
    :primaryButtonLabel="$t('buttons.install')"
    :closeOnSideClick="true"
  >
  <div class="column" style="padding: 0 30px; align-items: flex-start; max-width: 500px;">
    <div style="width: 100%; text-align: center; font-weight: 600; font-size: 27px; margin-bottom: 25px">
      {{ $t("dialogs.guiUpdate.title") }}
    </div>
    <div style="margin-bottom: 15px;">
      {{ $t("dialogs.guiUpdate.mainText") }}:
    </div>
    <div>
      <span style="font-weight: bold; margin-right: 15px;">{{ $t("dialogs.guiUpdate.version") }}:</span>{{ selectedGuiUpdate ? selectedGuiUpdate.version : "loading..." }}
    </div>
    <div style="font-weight: bold;">
      {{ $t("dialogs.guiUpdate.changelog") }}:
    </div>
    <div style="background: rgb(217,217,217); border-radius: 8px; padding: 10px; width: 480px; min-height: 100px; max-height: 200px; overflow-y: auto; margin-top: 5px; white-space: pre-wrap;">
      {{ selectedGuiUpdate ? selectedGuiUpdate.changelog : "loading..." }}
    </div>
    <div style="margin-top: 20px;">
      {{ $t("dialogs.guiUpdate.question") }}
    </div>
  </div>

  </HCGenericDialog>

  <HCLoading ref="downloading" :text="loadingText" />

  <HCSnackbar
    :labelText="errorText"
    ref="snackbar"
  ></HCSnackbar>


  <div
    style="
      display: flex;
      flex: 1;
      flex-direction: column;
      margin-bottom: 80px;
      padding: 0 30px;
      width: 70%;
      align-items: center;
      min-width: 900px;
    "
  >
    <div
      class="row"
      style="
        width: 100%;
        justify-content: flex-end;
        align-items: center;
        max-width: 1100px;
        margin-top: 20px;
        margin-bottom: -5px;
      "
    >
      <HCSelectCard
        style="
          width: 200px;
          margin-right: 5px;
          box-shadow: 0 0px 3px -1px #9b9b9b;
          --hc-label-background: #e8e8eb;
        "
        :placeholder="$t('main.holochainVersions')"
        :items="holochainVersionOptions"
        @item-selected="selectedHolochainVersion = $event"
      ></HCSelectCard>
      <img
        src="/img/Square284x284Logo.png"
        style="
          height: 30px;
          filter: grayscale(50%);
          margin-right: 20px;
          margin-left: -2px;
        "
      />

      <HCSelectCard
        style="
          width: 200px;
          margin-right: 5px;
          box-shadow: 0 0px 3px -1px #9b9b9b;
          --hc-label-background: #e8e8eb;
        "
        :placeholder="$t('main.sortBy')"
        :items="sortOptions"
        @item-selected="sortOption = $event"
      ></HCSelectCard>
      <mwc-icon style="color: #482edf; text-shadow: 0 0px 5px #9b9b9b"
        >sort</mwc-icon
      >
    </div>

    <!-- Web Apps -->
    <div
      class="row section-title"
      :class="{ borderBottomed: showWebApps }"
      style="margin-top: -18px"
    >
      <span
        style="margin-left: 10px; font-size: 23px; color: rgba(0, 0, 0, 0.6)"
        :title="$t('main.webAppsHelper')"
        >{{ $t("main.webApps") }}</span
      >
      <span
        @click="showWebApps = !showWebApps"
        class="show-hide"
        style="opacity: 0.7; cursor: pointer; margin-left: 10px"
      >
        {{ showWebApps ? "[-]" : "[show]" }}
      </span>
    </div>
    <div v-if="showWebApps" style="margin-bottom: 50px; width: 100%">
      <div
        v-if="noWebApps"
        style="margin-top: 30px; color: rgba(0, 0, 0, 0.6); text-align: center"
      >
        {{ $t("main.noWebApps") }}
        {{
          selectedHolochainVersion === "All Versions"
            ? "."
            : " in this Holochain Version."
        }}
      </div>

      <div
        v-else
        v-for="app in sortedApps"
        :key="app.webAppInfo.installed_app_info.installed_app_id"
        style="
          display: flex;
          flex-direction: column;
          width: 100%;
          align-items: center;
        "
      >
        <InstalledAppCard
          v-if="app.webAppInfo.web_uis.default.type !== 'Headless'"
          style="margin: 5px; display: flex; flex: 1"
          :app="app"
          @openApp="$emit('openApp', $event)"
          @uninstallApp="$emit('uninstall-app', $event)"
          @disableApp="$emit('disable-app', $event)"
          @enableApp="$emit('enable-app', $event)"
          @startApp="$emit('startApp', $event)"
          @updateGui="openUpdateGuiDialog($event)"
        />
      </div>
    </div>

    <!-- Headless Apps -->
    <div
      class="row section-title"
      :class="{ borderBottomed: showHeadlessApps }"
    >
      <span
        style="margin-left: 10px; font-size: 23px; color: rgba(0, 0, 0, 0.6)"
        :title="$t('main.headlessAppsHelper')"
        >{{ $t("main.headlessApps") }}</span
      >
      <span
        @click="showHeadlessApps = !showHeadlessApps"
        class="show-hide"
        style="opacity: 0.7; cursor: pointer; margin-left: 10px"
      >
        {{ showHeadlessApps ? "[-]" : "[show]" }}
      </span>
    </div>
    <div v-if="showHeadlessApps" style="margin-bottom: 50px; width: 100%">
      <div
        v-if="noHeadlessApps"
        style="margin-top: 30px; color: rgba(0, 0, 0, 0.6); text-align: center"
      >
        {{ $t("main.noHeadlessApps") }}
        {{
          selectedHolochainVersion === "All Versions"
            ? "."
            : " in this Holochain Version."
        }}
      </div>
      <div
        v-for="app in sortedApps"
        :key="app.webAppInfo.installed_app_info.installed_app_id"
        style="
          display: flex;
          flex-direction: column;
          width: 100%;
          align-items: center;
        "
      >
        <InstalledAppCard
          v-if="app.webAppInfo.web_uis.default.type === 'Headless'"
          style="margin: 5px; display: flex; flex: 1"
          :app="app"
          @openApp="$emit('openApp', $event)"
          @uninstallApp="$emit('uninstall-app', $event)"
          @disableApp="$emit('disable-app', $event)"
          @enableApp="$emit('enable-app', $event)"
        />
      </div>
    </div>

    <!-- Holochain verison info -->
    <div
      class="row section-title"
      :class="{ borderBottomed: showHolochainVersions }"
    >
      <span
        style="margin-left: 10px; font-size: 23px; color: rgba(0, 0, 0, 0.6)"
        :title="$t('main.holochainVersionsHelper')"
        >{{ $t("main.holochainVersions") }}</span
      >
      <span
        @click="showHolochainVersions = !showHolochainVersions"
        class="show-hide"
        style="opacity: 0.7; cursor: pointer; margin-left: 10px"
      >
        {{ showHolochainVersions ? "[-]" : "[show]" }}
      </span>
      <span style="flex: 1"></span>
      <span
        @click="refreshStorageInfo"
        style="margin-right: 5px; margin-bottom: -8px; cursor: pointer"
      >
        <img
          src="/img/refresh.png"
          style="height: 12px; margin-right: 3px; opacity: 0.7"
        />
        {{ $t("main.refresh") }}
      </span>
    </div>
    <div
      v-if="showHolochainVersions"
      class="column"
      style="margin-bottom: 50px; width: 100%; align-items: center"
    >
      <div
        v-if="noHolochainVersions"
        style="margin-top: 30px; color: rgba(0, 0, 0, 0.6); text-align: center"
      >
        {{ $t("main.noHolochainVersions") }}
      </div>
      <div v-else style="max-width: 1090px; width: 99%">
        <div
          v-for="hcVersion in holochainVersions"
          :key="hcVersion"
          style="
            display: flex;
            flex: 1;
            flex-direction: column;
            width: 100%;
            align-items: center;
          "
        >
          <div class="row hc-version" style="margin: 5px 0">
            <img
              src="/img/Square284x284Logo.png"
              style="height: 42px; margin-left: 11px; margin-right: 11px"
            />
            <div style="font-weight: 600; font-size: 1.1em">
              {{ hcVersion }}
            </div>
            <span style="display: flex; flex: 1"></span>
            <span
              v-if="storageInfos && !refreshing"
              style="font-weight: 600; margin-right: 15px"
              >{{ totalStorageString(hcVersion) }}</span
            >
            <StackedChart
              v-if="storageInfos && !refreshing"
              :fractions="storageFractions(hcVersion)"
              :labels="storageLabels(hcVersion)"
              style="width: 200px; height: 34px; margin-right: 12px"
            ></StackedChart>
            <!-- <span style="width: 120px; text-align: center">{{
              storageInfos[hcVersion]
                ? prettyBytes(storageInfos[hcVersion].conductor)
                : "?"
            }}</span>
            <span style="width: 120px; text-align: center">{{
              storageInfos[hcVersion]
                ? prettyBytes(storageInfos[hcVersion].uis)
                : "?"
            }}</span> -->
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType } from "vue";
import { uniq } from "lodash-es";

import "@material/mwc-button";
import "@material/mwc-icon-button";
import "@material/mwc-icon";

import { HolochainAppInfo, HolochainAppInfoExtended, HolochainId, StorageInfo } from "../types";
import { getCellId, isAppRunning } from "../utils";
import InstalledAppCard from "./InstalledAppCard.vue";
import HCSelectCard from "./subcomponents/HCSelectCard.vue";
import StackedChart from "./subcomponents/StackedChart.vue";
import HCGenericDialog from "./subcomponents/HCGenericDialog.vue";
import HCLoading from "./subcomponents/HCLoading.vue";
import { invoke } from "@tauri-apps/api/tauri";
import prettyBytes from "pretty-bytes";
import HCSnackbar from "./subcomponents/HCSnackbar.vue";
import { devhubCells, getHappReleasesByEntryHashes, fetchGui } from "../devhub/get-happs";
import { AppInfo, AppWebsocket, decodeHashFromBase64, encodeHashToBase64, EntryHash } from "@holochain/client";
import { GUIReleaseEntry, HappReleaseEntry } from "../devhub/types";
import { ActionTypes } from "../store/actions";


export default defineComponent({
  name: "InstalledAppsList",
  components: {
    InstalledAppCard,
    HCSelectCard,
    StackedChart,
    HCGenericDialog,
    HCLoading,
    HCSnackbar,
  },
  props: {
    installedApps: {
      type: Object as PropType<Array<HolochainAppInfo>>,
      required: true,
    },
  },
  data(): {
    appWebsocket: AppWebsocket | undefined;
    devhubAppInfo: AppInfo | undefined;
    sortOptions: [string, string][];
    sortOption: string | undefined;
    selectedHolochainVersion: string;
    showHeadlessApps: boolean;
    showWebApps: boolean;
    showHolochainVersions: boolean;
    storageInfos: Record<string, StorageInfo>;
    refreshing: boolean;
    refreshTimeout: number | null;
    extendedAppInfos: Array<HolochainAppInfoExtended> | undefined;
    selectedApp: HolochainAppInfoExtended | undefined;
    selectedGuiUpdate: GUIReleaseEntry | undefined;
    selectedGuiUpdateHash: EntryHash | undefined;
    loadingText: string;
    errorText: string;
  } {
    return {
      appWebsocket: undefined,
      devhubAppInfo: undefined,
      sortOptions: [
        ["name", "name"],
        ["name descending", "name descending"],
        // ["Holochain Version", "Holochain Version"],
      ],
      sortOption: undefined,
      selectedHolochainVersion: "All Versions",
      showHeadlessApps: true,
      showWebApps: true,
      showHolochainVersions: true,
      storageInfos: {},
      refreshing: false,
      refreshTimeout: null,
      extendedAppInfos: undefined,
      selectedApp: undefined,
      selectedGuiUpdate: undefined,
      selectedGuiUpdateHash: undefined,
      loadingText: "",
      errorText: "Unknown error occured",
    };
  },
  emits: ["openApp", "uninstall-app", "enable-app", "disable-app", "startApp"],
  async mounted() {
    await Promise.all(
      this.installedApps.map(async (app) => {
        this.storageInfos[app.holochainVersion] = await invoke(
          "get_storage_info",
          { holochainId: app.holochainId }
        );
      })
    );

    const holochainId = this.$store.getters["holochainIdForDevhub"];
    // connect to AppWebsocket
    const port = this.$store.getters["appInterfacePort"](holochainId);
    const appWebsocket = await AppWebsocket.connect(`ws://localhost:${port}`, 40000);
    this.appWebsocket = appWebsocket;
    const devhubAppInfo = await appWebsocket.appInfo({
        installed_app_id: `DevHub-${holochainId.content}`,
    });
    this.devhubAppInfo = devhubAppInfo;

    await this.checkForUiUpdates();

  },
  computed: {
    sortedApps() {
      // if extended happ releases are not yet fetched from the DevHub to include potential
      // GUI updates, just return installedApps with guiUpdateAvailable undefined
      let sortedAppList: Array<HolochainAppInfoExtended> = this.extendedAppInfos ? this.extendedAppInfos : this.installedApps.map((app) => {
        return {
          webAppInfo: app.webAppInfo,
          holochainId: app.holochainId,
          holochainVersion: app.holochainVersion,
          guiUpdateAvailable: undefined,
        }
      });

      if (this.selectedHolochainVersion !== "All Versions") {
        sortedAppList = sortedAppList.filter(
          (app) => app.holochainVersion === this.selectedHolochainVersion
        );
      }

      if (this.sortOption === "name") {
        sortedAppList = sortedAppList.sort((appA, appB) =>
          appA.webAppInfo.installed_app_info.installed_app_id.localeCompare(
            appB.webAppInfo.installed_app_info.installed_app_id
          )
        );
      } else if (this.sortOption === "name descending") {
        sortedAppList = sortedAppList.sort((appA, appB) =>
          appB.webAppInfo.installed_app_info.installed_app_id.localeCompare(
            appA.webAppInfo.installed_app_info.installed_app_id
          )
        );
      } else {
        // default is alphabetical by app id
        sortedAppList = sortedAppList.sort((appA, appB) =>
          appA.webAppInfo.installed_app_info.installed_app_id.localeCompare(
            appB.webAppInfo.installed_app_info.installed_app_id
          )
        );
      }

      return sortedAppList;
    },
    noHeadlessApps(): boolean {
      return !this.sortedApps.some(
        (app) => app.webAppInfo.web_uis.default.type === "Headless"
      );
    },
    noWebApps(): boolean {
      return this.sortedApps.every(
        (app) => app.webAppInfo.web_uis.default.type === "Headless"
      );
    },
    noHolochainVersions(): boolean {
      return this.noWebApps && this.noHeadlessApps;
    },
    holochainVersions(): string[] {
      const allApps = this.installedApps;
      return uniq(allApps.map((app) => app.holochainVersion));
    },
    holochainVersionOptions(): [string, string][] {
      let allApps = this.installedApps;
      let hcVersions: [string, string][] = [["All Versions", "All Versions"]];
      uniq(allApps.map((app) => app.holochainVersion)).forEach((hcVer) => {
        hcVersions.push([hcVer, hcVer]);
      });
      return hcVersions;
    },
  },
  methods: {
    prettyBytes,
    isAppRunning,
    isAppHeadless(app: HolochainAppInfo) {
      return app.webAppInfo.web_uis.default.type === "Headless";
    },
    async checkForUiUpdates() {

      // check for GUI updates
      const allApps: Array<HolochainAppInfo> = this.$store.getters["allApps"];
      const allHappReleaseHashes = allApps.map((app) => app.webAppInfo.happ_release_hash ? decodeHashFromBase64(app.webAppInfo.happ_release_hash) : undefined);
      // console.log("@InstalledAppsList: allHappReleaseHashes from store's allApps: ", allHappReleaseHashes);
      const happReleases: Array<HappReleaseEntry | undefined> = await getHappReleasesByEntryHashes((this.appWebsocket! as AppWebsocket), this.devhubAppInfo!, allHappReleaseHashes);

      // console.log("@InstalledAppsList: happReleaseHashes: ", happReleases);

      // compare with existing

      const extendedAppInfos: Array<HolochainAppInfoExtended> = allApps.map((appInfo: HolochainAppInfo, idx) => {

        const isGuiUpdateAvailable = (appInfo.webAppInfo.web_uis.default.type === "WebApp" && happReleases[idx]?.official_gui)
          ? appInfo.webAppInfo.web_uis.default.gui_release_hash != encodeHashToBase64(happReleases[idx]?.official_gui!)
          : false

        return {
          webAppInfo: appInfo.webAppInfo,
          holochainId: appInfo.holochainId,
          holochainVersion: appInfo.holochainVersion,
          guiUpdateAvailable: isGuiUpdateAvailable ? happReleases[idx]?.official_gui : undefined,
        }
      });

      // console.log("@InstalledAppsLlist: extendedAppInfos: ", extendedAppInfos);

      this.extendedAppInfos = extendedAppInfos;
    },
    async openUpdateGuiDialog(app: HolochainAppInfoExtended) {
      this.selectedApp = app;

      // console.log("Gui release hash @openUpdateGuiDialog: ", app.guiUpdateAvailable);
      (this.$refs.updateGuiDialog as typeof HCGenericDialog).open();

      if (this.appWebsocket && this.devhubAppInfo) {
          const cells = devhubCells(this.devhubAppInfo);
          const guiReleaseResponse = await this.appWebsocket?.callZome({
          cap_secret: null,
          cell_id: getCellId(cells.happs.find((c) => "provisioned" in c )!)!,
          fn_name: "get_gui_release",
          zome_name: "happ_library",
          payload: {
            id: app.guiUpdateAvailable,
          },
          provenance: getCellId(cells.happs.find((c) => "provisioned" in c )!)![1],
        });

        this.selectedGuiUpdate = guiReleaseResponse.payload.content;
        // console.log("Got GUI Release: ", guiReleaseResponse.payload.content);
      } else {
        alert!("Error: AppWebsocket or DevHub AppInfo undefined.")
        this.selectedGuiUpdate = undefined;
      }
    },
    storageFractions(holochainVersion: string) {
      const storageInfo: StorageInfo = this.storageInfos[holochainVersion];
      if (storageInfo) {
        const totalStorage = this.totalStorage(holochainVersion);
        const fractions = Object.values(storageInfo).map(
          (value: number) => (value / totalStorage!) * 100
        );
        return fractions;
      } else {
        return undefined;
      }
    },
    totalStorage(holochainVersion: string): number | undefined {
      const storageInfo = this.storageInfos[holochainVersion];
      if (storageInfo) {
        return Object.values(storageInfo).reduce(
          (acc, currValue) => acc + currValue
        );
      } else {
        return undefined;
      }
    },
    storageLabels(holochainVersion: string) {
      const storageInfo = this.storageInfos[holochainVersion];
      if (storageInfo) {
        return Object.entries(storageInfo).map(
          ([key, value]) => `${key} (${prettyBytes(value)})`
        );
      } else {
        return undefined;
      }
    },
    async refreshStorageInfo() {
      this.refreshing = true;
      this.refreshTimeout = window.setTimeout(
        () => (this.refreshing = false),
        200
      );
      await Promise.all(
        this.installedApps.map(async (app) => {
          this.storageInfos[app.holochainVersion] = await invoke(
            "get_storage_info",
            { holochainId: app.holochainId }
          );
        })
      );
    },
    totalStorageString(hcVersion: string) {
      const totalStorageBytes = this.totalStorage(hcVersion);
      if (totalStorageBytes) {
        return prettyBytes(totalStorageBytes);
      } else {
        return "?";
      }
    },
    async updateGui() {
      this.loadingText = "Connecting with DevHub";
      (this.$refs.downloading as typeof HCLoading).open();

      this.loadingText = "Downloading...";

      let bytes = undefined;

      try {
        bytes = await fetchGui(
          this.appWebsocket! as AppWebsocket,
          this.devhubAppInfo!,
          this.selectedGuiUpdate!.web_asset_id,
        );
      } catch (e) {
        console.error("Error fetching the UI: ", e);
        this.errorText = `Error fetching the UI: ${e}`;
        (this.$refs.snackbar as typeof HCSnackbar).show();
        (this.$refs.downloading as typeof HCLoading).close();
      }

      this.loadingText = "Installing...";

      if (bytes) {
        try {
          await invoke("update_default_ui", {
            holochainId: this.selectedApp!.holochainId,
            appId: this.selectedApp!.webAppInfo.installed_app_info.installed_app_id,
            uiZipBytes: bytes,
            guiReleaseHash: encodeHashToBase64(this.selectedApp!.guiUpdateAvailable!),
          });
          this.loadingText = "";
          (this.$refs.downloading as typeof HCLoading).close();
          (this.$refs.updateGuiDialog as typeof HCGenericDialog).close();
          this.selectedGuiUpdate = undefined;
          this.selectedGuiUpdateHash = undefined;

          // to remove the update button:
          await this.$store.dispatch(ActionTypes.fetchStateInfo);
          window.setTimeout(() => this.checkForUiUpdates(), 500);
        } catch (e) {
          console.error("Error updating the UI: ", e);
          this.errorText = `Error updating the UI: ${e}`;

          (this.$refs as any).snackbar.show();
          (this.$refs.downloading as typeof HCLoading).close();
          this.loadingText = "";
        }
      }
    },
  },
});
</script>

<style scoped>
.show-hide:hover {
  color: black;
}
.section-title {
  width: 98%;
  margin: 10px;
  max-width: 1080px;
  padding-bottom: 3px;
  align-items: center;
}

.borderBottomed {
  border-bottom: 2px solid rgba(0, 0, 0, 0.4);
}

.hc-version {
  align-items: center;
  flex: 1;
  width: 100%;
  max-width: 1090px;
  margin-top: 8px;
  border-radius: 15px;
  padding: 8px 0;
  background: white;
  box-shadow: 0 0px 5px #9b9b9b;
}
</style>
