<script lang="ts">
  import type { ArticleSnapshot, OpenAIRequest } from "../lib/utils";
  import { getSelection, toSummarySource } from "../lib/utils";
  import { onMount } from "svelte";
  import { writable, get } from "svelte/store";
  import TabNode from "./TabNode.svelte";

  const URL = "https://chatgpt.com/";
  let isSettingsVisible = false;
  let messages: { [key: string]: string } = {
    sidepanel_setting: "",
    sidepanel_maxCharstoSplit: "",
    sidepanel_buttonLabel: "",
    sidepanel_promptTemplate: "",
    sidepanel_autoSend: "",
    sidepanel_delete: "",
    sidepanel_addButton: "",
    sidepanel_save: "",
    sidepanel_close: "",
    sidepanel_capture: "",
  };

  const open = async (currenturl: string) => {
    const iframe = document.getElementById("preview") as HTMLIFrameElement;
    await chrome.declarativeNetRequest.updateSessionRules({
      removeRuleIds: [1],
      addRules: [
        {
          id: 1,
          priority: 1,
          action: {
            type: chrome.declarativeNetRequest.RuleActionType.MODIFY_HEADERS,
            responseHeaders: [
              {
                header: "x-frame-options",
                operation: chrome.declarativeNetRequest.HeaderOperation.REMOVE,
              },
              {
                header: "content-security-policy",
                operation: chrome.declarativeNetRequest.HeaderOperation.REMOVE,
              },
            ],
          },
          condition: {
            urlFilter: "*",
            resourceTypes: [
              chrome.declarativeNetRequest.ResourceType.MAIN_FRAME,
              chrome.declarativeNetRequest.ResourceType.SUB_FRAME,
              chrome.declarativeNetRequest.ResourceType.XMLHTTPREQUEST,
              chrome.declarativeNetRequest.ResourceType.WEBSOCKET,
            ],
          },
        },
      ],
    });
  };

  onMount(() => {
    open(URL);
    for (const key in messages) {
      messages[key] = chrome.i18n.getMessage(key);
    }
  });
  interface Tab {
    id: number;
    title: string;
    url: string;
    favIconUrl?: string;
    openerTabId?: number;
    children?: Tab[];
  }
  let tabTree: Tab[] = [];

  onMount(async () => {
    const tabs = await chrome.tabs.query({});
    const tabMap = new Map<number, Tab>();

    // すべてのタブをtabMapに格納
    tabs.forEach((tab) => {
      tabMap.set(tab.id!, {
        id: tab.id!,
        title: tab.title || "Untitled",
        url: tab.url || "",
        favIconUrl: tab.favIconUrl || "",
        openerTabId: tab.openerTabId,
      });
    });

    // タブの親子関係を構築
    tabs.forEach((tab) => {
      const currentTab = tabMap.get(tab.id!);
      if (
        currentTab &&
        tab.openerTabId !== undefined &&
        tabMap.has(tab.openerTabId)
      ) {
        const parentTab = tabMap.get(tab.openerTabId)!;
        if (!parentTab.children) {
          parentTab.children = [];
        }
        parentTab.children.push(currentTab);
      }
    });

    // ルートタブ（親がないタブ）をtabTreeに追加
    tabTree = Array.from(tabMap.values()).filter((tab) => !tab.openerTabId);
  });
</script>

<!-- チャットメッセージ表示エリア -->
<div class="panel">
  <div class="mainContent">
    <!-- svelte-ignore a11y-missing-attribute -->
    <div id="tree">
      <ul>
        {#each tabTree as tab}
          <TabNode {tab} />
        {/each}
      </ul>
    </div>
  </div>
</div>

<style>
  .panel {
    display: flex;
    flex-direction: column;
    height: 100vh;
    margin: 0;
  }
  .mainContent {
    flex: 1 1 auto; /*残りの領域全体を占有する*/
    /*padding: 20px;*/
    position: relative;
  }
</style>
