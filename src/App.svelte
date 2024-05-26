<script lang="ts">
  let containerElement: HTMLDivElement;
  let loginElement: HTMLInputElement;
  let passwordElement: HTMLInputElement;
  let isHidden = false;

  const t = (translationKey: string): string => {
    return chrome.i18n.getMessage(translationKey) ?? translationKey;
  };

  const goToNextLoginStep = () => {
    const nextStepButton = getButtonByTagName("button") as HTMLButtonElement;

    nextStepButton?.click();
  };

  const fillInput = (input: HTMLInputElement, newValue: string) => {
    const dispatchEvent = (eventName: string) =>
      input.dispatchEvent(new Event(eventName, { bubbles: true, cancelable: true }));

    input.value = newValue;

    dispatchEvent("input");
    dispatchEvent("keyup");
    dispatchEvent("change");
  };

  const onLoginFill = () => {
    const loginInput = getLoginInput() as HTMLInputElement;
    const login = loginElement.value;

    if (loginInput && login?.length > 0) {
      fillInput(loginInput, login);
      goToNextLoginStep();
    }
  };

  const onPasswordFill = () => {
    const passwordCharaters = passwordElement.value.split("");
    let success = false;

    const passwordInputs = getPasswordInputs();

    for (const passwordInput of passwordInputs) {
      if (passwordInput.disabled) continue;

      const id = passwordInput.id;
      const idParts = id.split("-");
      const idxStr = idParts[idParts.length - 1];
      const idx = Number(idxStr);

      fillInput(passwordInput, passwordCharaters[idx]);
      success = true;
    }

    if (success) {
      goToNextLoginStep();
      onPasswordFilled();
    }
  };

  const onPasswordFilled = () => {
    isHidden = true;

    const checkIfSuccessfullLoginInterval = 500;
    const checkIfSuccessfullLoginTimeout = 30 * checkIfSuccessfullLoginInterval;

    const interval = setInterval(() => {
      const contentElement = document.getElementsByTagName("pekao-dashboard");

      if (!!contentElement) {
        fillInput(loginElement, "");
        fillInput(passwordElement, "");
        clearInterval(interval);
      }
    }, checkIfSuccessfullLoginInterval);

    setTimeout(() => {
      clearInterval(interval);
    }, checkIfSuccessfullLoginTimeout);
  };

  const getLoginInput = (): HTMLInputElement | null => {
    const test = (e: Element) => {
      const tagName = e.localName;
      const tagId = e.id;

      return tagName === "input" && tagId == "customer";
    };

    return findElementRecursive(test, document.body) as HTMLInputElement;
  };

  const getPasswordInputs = (): HTMLInputElement[] => {
    const test = (e: Element) => {
      const tagId = e.id;
      const tagName = e.localName;
      const tagType = e.getAttribute("type");
      const tagIdParts = (tagId ?? "").split("-");

      return tagName === "input" && tagType == "password" && tagIdParts.length === 6;
    };

    return findElementsRecursive(test, document.body) as HTMLInputElement[];
  };

  const getButtonByTagName = (name: string): Element | null => {
    const test = (e: Element) => {
      const tagName = e.localName;
      const tagClasses = [...e.classList];

      return tagName === name && tagClasses.includes("button-primary");
    };

    return findElementRecursive(test, document.body);
  };

  const findElementRecursive = (test: (e: Element) => boolean, element: Element): Element | null => {
    if (test(element)) return element;

    for (const child of element.children) {
      const foundChild = findElementRecursive(test, child);

      if (foundChild) return foundChild;
    }

    for (const child of element.shadowRoot?.children ?? []) {
      const foundChild = findElementRecursive(test, child);

      if (foundChild) return foundChild;
    }

    return null;
  };

  const findElementsRecursive = (test: (e: Element) => boolean, element: Element): Element[] => {
    let result = [];

    for (const child of element.children) {
      const foundChildren = findElementsRecursive(test, child);

      result = [...result, ...foundChildren];
    }

    for (const child of element.shadowRoot?.children ?? []) {
      const foundChildren = findElementsRecursive(test, child);

      result = [...result, ...foundChildren];
    }

    if (test(element)) {
      result = [...result, element];
    }

    return result;
  };
</script>

<div bind:this={containerElement} class="mpf-drawer" class:mpf-hidden={isHidden}>
  <div class="mpf-background">
    <div class="mpf-wrapper">
      <div class="mpf-title">
        <h3 class="title">{t("fill_header_title")}</h3>
        <p class="description">{t("fill_header_description")}</p>
      </div>
      <input bind:this={loginElement} id="mpf-login-input" class="mpf-input" type="text" />
      <button class="mpf-button" on:click={onLoginFill}>{t("fill_login")}</button>
      <input bind:this={passwordElement} id="mpf-password-input" class="mpf-input" type="password" />
      <button class="mpf-button" on:click={onPasswordFill}>{t("fill_password")}</button>
    </div>
  </div>
  <button class="mpf-drawer-handle" on:click={() => (isHidden = !isHidden)}>{isHidden ? ">" : "<"}</button>
</div>

<style lang="scss">
  .mpf-drawer {
    position: fixed;
    top: 50%;
    transform: translateY(-50%);
    display: flex;
    transition: transform 0.3s ease-out;
  }

  .mpf-background {
    --pekao-color: #c6161d;
    display: flex;
    align-items: center;
    background-color: var(--pekao-color);
    border-radius: 0 20px 20px 0;
    z-index: 99998;
  }

  .mpf-drawer-handle {
    background: url("data:image/svg+xml,%3Csvg%20height%3D%22270%22%20viewBox%3D%220%200%2051%20270%22%20width%3D%2251%22%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20xmlns%3Axlink%3D%22http%3A%2F%2Fwww.w3.org%2F1999%2Fxlink%22%3E%3ClinearGradient%20id%3D%22a%22%20x1%3D%220%25%22%20x2%3D%22100%25%22%20y1%3D%220%25%22%20y2%3D%220%25%22%3E%3Cstop%20offset%3D%220%22%20stop-color%3D%22%23ac1419%22%2F%3E%3Cstop%20offset%3D%221%22%20stop-color%3D%22%23c6171d%22%2F%3E%3C%2FlinearGradient%3E%3Cpath%20d%3D%22m50.2311907%20271c-.0181405-2.254133-.044929-3.924839-.0803655-5.012118-2.2305426-68.438533-50.14613694-67.705242-50.14613694-109.797406%200-.443908-.01054858-50.883244%200-51.318123%201.00874619-41.5868944%2049.76371954-42.4200594%2050.22323364-103.99391325.0014505-.194159.0025401%2089.84636105.0032688%20270.12156025z%22%20fill%3D%22url%28%23a%29%22%20fill-rule%3D%22evenodd%22%20transform%3D%22matrix%28-1%200%200%201%2051%20-1%29%22%2F%3E%3C%2Fsvg%3E");
    width: 51px;
    height: 260px;
    margin: 20px 0 20px -1px;
    border: 0;
    align-self: center;
    color: white;
    font-weight: bold;
    font-size: 25px;
  }

  .mpf-hidden {
    transform: translate(calc(-100% + 50px), -50%);
  }

  .mpf-wrapper {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    padding: 2.5rem;
  }

  .mpf-title {
    color: #fff;
    white-space: nowrap;
    align-self: flex-start;
  }

  .mpf-input {
    margin: 2rem 0 1rem;
    box-sizing: border-box;
    box-shadow: none;
    appearance: none;
    width: 100%;
    height: 40px;
    line-height: 40px;
    padding: 4px 20px 0;
    font-size: 16px;
    font-weight: 400;
    color: #404040;
    background: #fff;
    border: 1px solid #8a8a8a;
    border-radius: 4px;
    height: 50px;
    line-height: 50px;
  }

  .mpf-button {
    appearance: none;
    height: 52px;
    padding: 0 24px;
    box-sizing: border-box;
    outline: none;
    font-size: 16px;
    font-weight: 700;
    line-height: 20px;
    font-family: Lato, sans-serif;
    text-align: center;
    transition:
      color 0.2s ease-in-out,
      background-color 0.2s ease-in-out,
      border-color 0.2s ease-in-out,
      box-shadow 0.2s ease-in-out;
    cursor: pointer;
    border-radius: 26px;
    white-space: normal;
    min-width: 140px;
    color: #d71920;
    background-color: #fff;
    border: 2px solid #f9c8ca;

    &:hover {
      color: #c51a20;
      border-color: #c51a20;
    }
  }
</style>
