<script>
  import { createEventDispatcher, onMount } from 'svelte';

  export let cards = [];
  export let compact = false;

  const dispatch = createEventDispatcher();

  let cardContainer;
  let AdaptiveCards;
  let Template;
  let isLibraryLoaded = false;
  let globalHostConfig;

  onMount(async() => {
    try {
      // 动态导入 AdaptiveCards 库
      const adaptiveCardsModule = await import('adaptivecards');
      const templatingModule = await import('adaptivecards-templating');

      AdaptiveCards = adaptiveCardsModule.default || adaptiveCardsModule;
      Template = templatingModule.Template;

      // 初始化 AdaptiveCards 全局设置
      AdaptiveCards.GlobalSettings.setTabIndexAtCardRoot = false;
      AdaptiveCards.GlobalSettings.enableFullJsonRoundTrip = true;

      // 设置主机配置
      setupHostConfig();

      // 注册自定义元素
      registerCustomElements();

      isLibraryLoaded = true;
      console.log('AdaptiveCards library loaded successfully');

      // 重新渲染现有卡片
      renderAllCards();
    } catch (error) {
      console.error('Failed to load AdaptiveCards library:', error);
      isLibraryLoaded = false;
    }
  });

  function setupHostConfig() {
    if (!AdaptiveCards) return;

    // Create the host config that will be used for individual card instances
    // In AdaptiveCards 3.x, we don't set a global host config
    globalHostConfig = new AdaptiveCards.HostConfig({
      fontFamily:
        'Microsoft YaHei, -apple-system, BlinkMacSystemFont, \'Segoe UI\', Roboto, sans-serif',
      fontSizes: {
        small: 12,
        default: 14,
        medium: 17,
        large: 21,
        extraLarge: 26
      },
      fontWeights: {
        lighter: 200,
        default: 400,
        bolder: 600
      },
      containerStyles: {
        default: {
          backgroundColor: '#FFFFFF',
          foregroundColors: {
            default: { default: '#333333', subtle: '#666666' },
            dark: { default: '#000000', subtle: '#333333' },
            light: { default: '#666666', subtle: '#999999' },
            accent: { default: '#007BFF', subtle: '#0056B3' },
            good: { default: '#28A745', subtle: '#1E7E34' },
            warning: { default: '#FFC107', subtle: '#E0A800' },
            attention: { default: '#DC3545', subtle: '#C82333' }
          }
        },
        emphasis: {
          backgroundColor: '#F8F9FA',
          foregroundColors: {
            default: { default: '#333333', subtle: '#666666' },
            dark: { default: '#000000', subtle: '#333333' },
            light: { default: '#666666', subtle: '#999999' },
            accent: { default: '#007BFF', subtle: '#0056B3' },
            good: { default: '#28A745', subtle: '#1E7E34' },
            warning: { default: '#FFC107', subtle: '#E0A800' },
            attention: { default: '#DC3545', subtle: '#C82333' }
          }
        }
      },
      spacing: {
        small: 2,
        default: 4,
        medium: 8,
        large: 12,
        extraLarge: 16,
        padding: 8
      },
      separator: {
        lineThickness: 1,
        lineColor: '#E9ECEF'
      },
      imageSizes: {
        small: 40,
        medium: 80,
        large: 160
      },
      actions: {
        maxActions: 9,
        spacing: 'small',
        buttonSpacing: 4,
        showCard: {
          actionMode: 'inline',
          inlineTopMargin: 8
        },
        orientation: 'vertical',
        actionAlignment: 'left'
      },
      adaptiveCard: {
        allowCustomStyle: true
      }
    });
  }

  function registerCustomElements() {
    if (!AdaptiveCards) return;

    // 注册ProgressBar自定义元素
    registerProgressBarElement();

  // 移除GameIcon自定义元素 - 避免与Action.Submit按钮重复
    // 所有游戏选择都应该使用标准的Action.Submit按钮
  }

  function registerProgressBarElement() {
    class ProgressBarElement extends AdaptiveCards.CardElement {
      static get typeName() {
        return 'ProgressBar';
      }

      constructor() {
        super();
        this.value = 0;
        this.max = 100;
        this.color = 'accent';
        this.showPercentage = true;
        this.animated = false;
      }

      getJsonTypeName() {
        return ProgressBarElement.typeName;
      }

      internalRender() {
        const container = document.createElement('div');
        container.className = 'progress-container';
        container.style.cssText = `
          width: 100%;
          margin: 8px 0;
        `;

        const progressBar = document.createElement('div');
        progressBar.className = 'progress-bar';
        progressBar.style.cssText = `
          width: 100%;
          height: 20px;
          background: rgba(255, 255, 255, 0.1);
          border-radius: 10px;
          overflow: hidden;
          position: relative;
        `;

        const progressFill = document.createElement('div');
        progressFill.className = 'progress-fill';
        const percentage = Math.min((this.value / this.max) * 100, 100);
        const colorMap = {
          accent: '#4FACFE',
          good: '#107C10',
          warning: '#FF8C00',
          attention: '#D83B01'
        };
        progressFill.style.cssText = `
          width: ${percentage}%;
          height: 100%;
          background: linear-gradient(90deg, ${colorMap[this.color] || colorMap['accent']}, ${this.lightenColor(colorMap[this.color] || colorMap['accent'], 20)});
          transition: width 0.3s ease;
          border-radius: 10px;
        `;

        if (this.showPercentage) {
          const percentageText = document.createElement('span');
          percentageText.className = 'progress-percentage';
          percentageText.textContent = `${Math.round(percentage)}%`;
          percentageText.style.cssText = `
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: white;
            font-size: 12px;
            font-weight: bold;
            z-index: 2;
          `;
          progressBar.appendChild(percentageText);
        }

        progressBar.appendChild(progressFill);
        container.appendChild(progressBar);

        return container;
      }

      lightenColor(color, percent) {
        const num = parseInt(color.replace('#', ''), 16);
        const amt = Math.round(2.55 * percent);
        const R = (num >> 16) + amt;
        const G = ((num >> 8) & 0x00ff) + amt;
        const B = (num & 0x0000ff) + amt;
        return (
          '#' +
          (
            0x1000000 +
            (R < 255 ? (R < 1 ? 0 : R) : 255) * 0x10000 +
            (G < 255 ? (G < 1 ? 0 : G) : 255) * 0x100 +
            (B < 255 ? (B < 1 ? 0 : B) : 255)
          )
            .toString(16)
            .slice(1)
        );
      }

      parse(source, context) {
        super.parse(source, context);
        this.value = source.value || 0;
        this.max = source.max || 100;
        this.color = source.color || 'accent';
        this.showPercentage = source.showPercentage !== false;
        this.animated = source.animated || false;
      }
    }

    AdaptiveCards.GlobalRegistry.elements.register(ProgressBarElement.typeName, ProgressBarElement);
  }

  // GameIcon 元素已移除 - 避免与 Action.Submit 按钮功能重复
  // 现在所有游戏选择都通过标准的 Action.Submit 按钮实现

  function renderCard(cardData) {
    if (!isLibraryLoaded || !AdaptiveCards) {
      return createLoadingCard();
    }

    try {
      // 解析卡片数据
      let cardPayload;
      if (typeof cardData === 'string') {
        try {
          cardPayload = JSON.parse(cardData);
        } catch (e) {
          cardPayload = {
            type: 'AdaptiveCard',
            version: '1.6',
            body: [
              {
                type: 'TextBlock',
                text: cardData,
                wrap: true
              }
            ]
          };
        }
      } else if (cardData.content) {
        if (typeof cardData.content === 'string') {
          try {
            cardPayload = JSON.parse(cardData.content);
          } catch (e) {
            cardPayload = {
              type: 'AdaptiveCard',
              version: '1.6',
              body: [
                {
                  type: 'TextBlock',
                  text: cardData.content,
                  wrap: true
                }
              ]
            };
          }
        } else {
          cardPayload = cardData.content;
        }
      } else if (cardData.adaptiveCard) {
        cardPayload = cardData.adaptiveCard;
      } else {
        cardPayload = cardData;
      }

      // 确保是有效的 AdaptiveCard 格式
      if (!cardPayload.type || cardPayload.type !== 'AdaptiveCard') {
        cardPayload = {
          type: 'AdaptiveCard',
          version: '1.6',
          body: [
            {
              type: 'TextBlock',
              text: JSON.stringify(cardPayload, null, 2),
              wrap: true
            }
          ]
        };
      }

      // 确保版本为 1.6
      cardPayload.version = '1.6';

      console.log('🎯 cardPayload.actionsOrientation:', cardPayload.actionsOrientation);

      // 若 cardPayload.actionsOrientation === 'vertical' 则将顶层 actions 转为 ActionSet 垂直显示
      if (cardPayload.actionsOrientation && cardPayload.actionsOrientation.toLowerCase() === 'vertical' && Array.isArray(cardPayload.actions)) {
        cardPayload.body = cardPayload.body || [];
        // 添加到 body 底部，保持文字在上、按钮在下的布局
        cardPayload.body.push({
          type: 'ActionSet',
          orientation: 'vertical',
          actions: cardPayload.actions
        });
        delete cardPayload.actions;
      }

      // 创建 AdaptiveCard 实例
      const adaptiveCard = new AdaptiveCards.AdaptiveCard();

      // 设置主机配置 (在 AdaptiveCards 3.x 中，我们在每个实例上设置 hostConfig)
      if (globalHostConfig) {
        adaptiveCard.hostConfig = globalHostConfig;
      }

      // 解析卡片
      adaptiveCard.parse(cardPayload);


      // 设置动作处理器
      adaptiveCard.onExecuteAction = action => {
        const actionData = {
          cardId: Date.now(),
          type: action.getJsonTypeName(),
          title: action.title,
          data: action.data || {},
          originalAction: action
        };

        handleCardAction(actionData);
      };

      // 渲染卡片
      const renderedCard = adaptiveCard.render();


      // 应用自定义样式
      if (renderedCard) {
        renderedCard.className = compact ? 'adaptive-card compact' : 'adaptive-card';

        if (compact) {
          // 紧凑模式样式（用于输入辅助卡片）
          renderedCard.style.cssText = `
            background: rgba(255, 255, 255, 0.95);
            border: 1px solid rgba(0, 0, 0, 0.1);
            border-radius: 8px;
            padding: 8px 12px;
            margin: 0;
            color: #333;
            width: 100%;
            font-family: "Microsoft YaHei", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            font-size: 0.9em;
          `;
        } else {
          // 标准模式样式（用于右侧面板卡片）
          renderedCard.style.cssText = `
              background: rgba(255, 255, 255, 0.1);
              backdrop-filter: blur(10px);
              border: 1px solid rgba(255, 255, 255, 0.2);
              border-radius: 12px;
              padding: 12px;
              margin-bottom: 16px;
              color: white;
              width: 100%;
              max-width: 100%;
              box-sizing: border-box;
              font-family: "Microsoft YaHei", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            `;
        }

      // GameIcon 事件处理已移除 - 现在只使用标准的 Action.Submit 按钮
      }

      return renderedCard;
    } catch (error) {
      console.error('Failed to render AdaptiveCard:', error);
      return createErrorCard(cardData, error.message);
    }
  }

  function createLoadingCard() {
    // 返回空的div，不显示任何加载内容
    const emptyDiv = document.createElement('div');
    emptyDiv.className = 'empty-loading-card';
    emptyDiv.style.cssText = `
      display: none;
    `;
    return emptyDiv;
  }

  function createErrorCard(cardData, errorMessage) {
    const errorDiv = document.createElement('div');
    errorDiv.className = 'card-error';
    errorDiv.style.cssText = `
      background: rgba(255, 100, 100, 0.2);
      border: 1px solid rgba(255, 100, 100, 0.4);
      border-radius: 12px;
      padding: 16px;
      margin-bottom: 16px;
      color: white;
      width: fit-content;
      min-width: 250px;
      max-width: 450px;
    `;

    errorDiv.innerHTML = `
      <div style="display: flex; align-items: center; gap: 8px; margin-bottom: 12px; font-weight: 600;">
        <span style="font-size: 1.2em;">⚠️</span>
        卡片渲染错误
      </div>
      <div style="margin-bottom: 8px; font-size: 0.9em; opacity: 0.8;">
        错误信息: ${errorMessage}
      </div>
      <details style="margin-top: 12px;">
        <summary style="cursor: pointer; font-size: 0.9em; opacity: 0.8;">查看原始数据</summary>
        <pre style="margin-top: 8px; padding: 8px; background: rgba(0,0,0,0.2); border-radius: 4px; font-size: 0.8em; overflow-x: auto;">${JSON.stringify(cardData, null, 2)}</pre>
      </details>
    `;

    return errorDiv;
  }

  // 防抖机制：防止短时间内重复触发
  let lastActionTime = 0;
  let lastActionId = '';
  const DEBOUNCE_TIME = 1000; // 1秒防抖

  function handleCardAction(action) {
    // 直接处理动作，不显示视觉反馈
    try {
      dispatch('cardAction', {
        cardId: action.cardId || 'unknown',
        action: action
      });
    } catch (error) {
      console.error('Error dispatching cardAction:', error);
    }
  }

  function renderAllCards() {
    if (!cardContainer || !isLibraryLoaded) {
      return;
    }

    // 清理现有卡片（GameIcon清理已移除，现在只使用标准按钮）

    // 清空容器
    cardContainer.innerHTML = '';

    // 渲染所有卡片
    cards.forEach((cardData, index) => {
      const renderedCard = renderCard(cardData);
      if (renderedCard) {
        cardContainer.appendChild(renderedCard);
      }
    });
  }

  // 响应式更新卡片 - 明确依赖 cards 数组
  $: if (isLibraryLoaded && cardContainer && cards) {
    renderAllCards();
  }
</script>

<div class="adaptive-card-panel" bind:this={cardContainer}>
  <!-- 移除加载提示，保持空白 -->
</div>

<style>
  .adaptive-card-panel {
    width: 100%;
    min-width: auto;
    max-width: none;
    max-height: none;
    overflow: visible;
    padding: 0;
  }

  .loading-indicator {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    padding: 16px;
    color: rgba(255, 255, 255, 0.8);
    font-size: 0.95em;
  }

  .spinner {
    width: 20px;
    height: 20px;
    border: 2px solid rgba(255, 255, 255, 0.3);
    border-top: 2px solid white;
    border-radius: 50%;
    animation: spin 1s linear infinite;
  }

  @keyframes spin {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }

  /* 全局样式用于渲染的卡片 */
  :global(.adaptive-card) {
    user-select: text;
    width: 100% !important;
    max-width: none !important;
    overflow: visible !important;
  }

  /* AdaptiveCard Actions容器 - 支持不同布局 */
  :global(.adaptive-card .ac-actions) {
    width: 100% !important;
    max-width: none !important;
    overflow: visible !important;
  }

  /* AdaptiveCard 按钮样式 */
  :global(.adaptive-card .ac-pushButton) {
    background: #007bff !important;
    border: 1px solid #007bff !important;
    color: white !important;
    border-radius: 4px !important;
    width: 120px !important;
    min-width: 120px !important;
    max-width: 120px !important;
    height: 36px !important;
    text-align: center !important;
    display: inline-flex !important;
    align-items: center !important;
    justify-content: center !important;
    box-sizing: border-box !important;
    white-space: nowrap !important;
    overflow: hidden !important;
    text-overflow: ellipsis !important;
    padding: 8px 12px !important;
    cursor: pointer !important;
    font-weight: 500 !important;
    font-size: 13px !important;
    margin: 2px !important;
  }

  /* 移除按钮焦点轮廓 */
  :global(.adaptive-card .ac-pushButton:focus) {
    outline: none !important;
    box-shadow: none !important;
  }

  :global(.adaptive-card .ac-pushButton:hover) {
    background: #0056b3 !important;
    border: 1px solid #0056b3 !important;
  }

  :global(.adaptive-card .ac-pushButton:active) {
    background: #004085 !important;
  }


  /* AdaptiveCard 输入框样式 */
  :global(.adaptive-card .ac-textInput),
  :global(.adaptive-card .ac-numberInput),
  :global(.adaptive-card .ac-choiceSetInput) {
    background: #ffffff !important;
    border: 1px solid #ced4da !important;
    color: #333333 !important;
    border-radius: 4px !important;
    padding: 8px 12px !important;
    transition: all 0.2s ease !important;
    margin: 4px 0 !important;
    width: 100% !important;
    box-sizing: border-box !important;
    font-size: 14px !important;
  }

  :global(.adaptive-card .ac-textInput:focus),
  :global(.adaptive-card .ac-numberInput:focus) {
    border-color: #007bff !important;
    outline: none !important;
    box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.25) !important;
  }

  :global(.adaptive-card .ac-textInput::placeholder),
  :global(.adaptive-card .ac-numberInput::placeholder) {
    color: #6c757d !important;
  }

  /* AdaptiveCard 文本样式 */
  :global(.adaptive-card .ac-textBlock) {
    color: #333333 !important;
    line-height: 1.5 !important;
    width: 100% !important;
    max-width: none !important;
    overflow: visible !important;
    white-space: normal !important;
    word-wrap: break-word !important;
  }

  /* AdaptiveCard 容器样式 */
  :global(.adaptive-card .ac-container) {
    background: #ffffff !important;
    border-radius: 8px !important;
    border: 1px solid #e9ecef !important;
  }

  /* AdaptiveCard 列样式 */
  :global(.adaptive-card .ac-column) {
    background: transparent !important;
    border-radius: 0 !important;
  }

  /* AdaptiveCard 图片样式 */
  :global(.adaptive-card .ac-image) {
    border-radius: 8px !important;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2) !important;
  }

  /* 自定义元素样式增强 */
  :global(.progress-container) {
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
  }

  /* GameIcon 相关样式已移除 - 现在只使用 Action.Submit 按钮 */
</style>
