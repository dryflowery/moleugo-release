<script>
    import Footer from "../component/Footer.svelte";
    import Header from "../component/Header.svelte";
    import {isListVisible } from "../lib/store"
    import {isLogin } from "../lib/store"
    import { push } from "svelte-spa-router";

    let userName = "";
    let isLengthValid = false;
    let isSpecialCharValid = false;
    let showResult = false; // 결과 메시지 표시 여부
    let saveMessage = ""; // 저장 메세지
    let savedUserName = ""; // 저장된 이름

    let usedNames = ["컴붕이1"]; // 이미 존재하는 닉네임들 (임의로 추가)

    let userPassword = "comBung@369"; //기존 비밀번호

    let inputOldPassword = ""; // 기존 비밀번호 입력값
    let isOldPasswordVisible = false; // 기존 비밀번호 표시 상태

    let inputNewPassword = ""; // 새 비밀번호 입력값
    let isNewPasswordVisible = false; // 새 비밀번호 표시 상태

    let inputNewPassword_verify = ""; // 새 비밀번호 확인용 입력값
    let isNewPasswordVisible_verify = false; // 새 비밀번호 확인용 표시 상태

    let passwordChangeMessage = ""; // 비밀번호 변경 결과 메시지
    let oldPasswordStatusColor = "";
    let oldPasswordIcon = "checkmark-outline";

    let oldPasswordStatus = "";

    let condition1Met = false; // 영문/숫자/특수문자 중 2가지 이상 포함
    let condition2Met = false; // 8자 이상 32자 이하 입력 (공백 제외)
    let condition3Met = false; // 연속 3자 이상 동일한 문자/숫자 제외
    
    let verifyPasswordMessage = "확인을 위해 새 비밀번호를 다시 입력하세요"; // 확인 메시지
    let verifyPasswordMessageStyle = ""; // 메시지 스타일

    let ispasswordCheckIcon = false // 비밀번호 설정 옆 아이콘 유/무
    

    function checkOldPassword() {
        if (inputOldPassword === userPassword) {
            oldPasswordStatus = "비밀번호가 일치합니다";
            oldPasswordStatusColor = "#238636";
            oldPasswordIcon = "checkmark-outline";
            saveNewPassword(); // 비밀번호가 일치하면 새 비밀번호 저장 함수 호출
        } else {
            oldPasswordStatus = "비밀번호가 일치하지 않습니다";
            oldPasswordStatusColor = "rgb(173, 44, 44)";
            oldPasswordIcon = "close-outline";
        }
    }

    // 비밀번호 조건 확인 함수
    function validateNewPassword(password) {
      // 1. 영문/숫자/특수문자 중 2가지 이상 포함
      condition1Met = [
        /[a-zA-Z]/.test(password), // 영문
        /[0-9]/.test(password), // 숫자
        /[^a-zA-Z0-9\s]/.test(password), // 특수문자
      ].filter(Boolean).length >= 2;

      // 2. 8자 이상 32자 이하 입력 (공백 제외)
      condition2Met = password.replace(/\s/g, "").length >= 8 && password.length <= 32;

      // 3. 연속 3자 이상 동일한 문자/숫자 제외
      condition3Met = !/(.)\1\1/.test(password);

      // 모든 조건 충족 여부 반환
      return condition1Met && condition2Met && condition3Met;
    }

      // 비밀번호 변경 버튼 클릭
      function saveNewPassword() {
        // 초기화
        passwordChangeMessage = "";

        // 기존 비밀번호 확인
        if (inputOldPassword !== userPassword) {
          return;
        }

        // 새 비밀번호 조건 확인
        if (!validateNewPassword(inputNewPassword)) {
          return;
        }

        // 새 비밀번호와 확인 비밀번호 일치 확인
        if (inputNewPassword !== inputNewPassword_verify) {
          return;
        }

        // 모든 조건 충족 시 비밀번호 변경
        userPassword = inputNewPassword;
        alert("새 비밀번호가 성공적으로 저장되었습니다.");
        ispasswordCheckIcon = true;
        
        // 입력값 초기화
        inputOldPassword = "";
        inputNewPassword = "";
        inputNewPassword_verify = "";
    }

    // 닉네임 검증 함수
    function validateUserName(name) {
        const lengthRegex = /^.{2,10}$/; // 2~10 글자
        const specialCharRegex = /^[^\s!@#$%^&*(),.?":{}|<>]*$/; // 띄어쓰기 및 특수문자 제외

        isLengthValid = lengthRegex.test(name);
        isSpecialCharValid = specialCharRegex.test(name);
    }

    // 닉네임 저장 버튼 클릭 이벤트
    function saveUserName() {
      showResult = true; // 결과 메시지 표시
      saveMessage = " "; // 기본값 설정

      if (isLengthValid && isSpecialCharValid) {
        if (usedNames.includes(userName)) {
          saveMessage = "중복된 닉네임입니다. 다시 설정해주세요.";
          } else {
            savedUserName = userName; // 저장된 닉네임 업데이트
            usedNames.push(userName); // 새 닉네임 추가
            saveMessage = "중복확인 닉네임 변경 완료";
          }
      } else if (!usedNames.includes(userName)) {
        saveMessage = "조건을 충족 해주세요";
      }
    }

    function toggleOldPasswordVisibility() { // OLD
        isOldPasswordVisible = !isOldPasswordVisible;
    }

    function toggleNewPasswordVisibility() { // NEW
        isNewPasswordVisible = !isNewPasswordVisible;
    }

    function toggleNewPasswordVisibility_verify() { // NEW
      isNewPasswordVisible_verify = !isNewPasswordVisible_verify;
    }

    // 닉네임이 변경될 때마다 검증
    $: validateUserName(userName);

    // 비밀번호 변경 시 조건 확인
    $: validateNewPassword(inputNewPassword);
      // 비밀번호 확인 일치 여부 검사
      $: {
      if (inputNewPassword_verify === "") {
        verifyPasswordMessage = "확인을 위해 새 비밀번호를 다시 입력하세요";
        verifyPasswordMessageStyle = ""; // 기본 스타일
      } else if (inputNewPassword_verify === inputNewPassword) {
        verifyPasswordMessage = "비밀번호가 일치합니다";
        verifyPasswordMessageStyle = "color: #238636;"; // 초록색
      } else {
        verifyPasswordMessage = "비밀번호가 일치하지 않습니다";
        verifyPasswordMessageStyle = "color: rgb(173, 44, 44);"; // 빨간색
      }
    }


  </script>
  
<main>
  <div class="header-container">
    <Header />
  </div>
  
  <div class="main-container" style="transform: translateX({$isListVisible ? 360 : 155}px); margin-right: {$isListVisible ? 360: 155}px; "> <!--알고리즘 리스트 유무에 따른 위치,크기 조절-->
     
    <div class="content">
        
      <div id="content-box">

          <div id="left-container">

            <!--프로필 사진 및 닉네임 변경 영역-->
            <div id="profile-edit-container">
              <!--프로필 사진 영역-->
              <div id="profile-image">
                <button id="profile-edit-Btn">설정</button>
              </div>

              <div id='change-userName-container'>

                <div id="change-userName-Title">
                  <t style="color: #bbbbbb;">닉네임: {savedUserName || "컴붕이1"} </t>
                  {#if saveMessage == '중복된 닉네임입니다. 다시 설정해주세요.'}
                    <ion-icon name="warning-outline" style="color: yellow;"></ion-icon>
                  {:else if saveMessage == "중복확인 닉네임 변경 완료"}
                    <ion-icon name="checkmark-outline" style="color: green;" ></ion-icon>
                  {/if}
                </div>

                <div id="change-userName">
                  <input type="text" placeholder="닉네임 변경 ..." bind:value="{userName}">
                  <button id="userName-edit-Btn" on:click={saveUserName}>변경</button>
                </div>

                <div id="verify-userName-container">

                  <div id="Requirements_1" 
                       class:valid={isLengthValid}  
                       class:invalid={!isLengthValid}> 
                       <ion-icon name="checkmark-outline"></ion-icon> 
                       2 ~ 10 글자 사이의 닉네임을 정해주세요
                  </div>

                  <div id="Requirements_2" 
                       class:valid={isSpecialCharValid}
                       class:invalid={!isSpecialCharValid}> 
                       <ion-icon name="checkmark-outline"></ion-icon> 
                       띄어쓰기를 포함한 특수문자 X
                  </div>

                  {#if showResult}
                    <div id="Requirements_3" 
                      class:success={saveMessage === "중복확인 닉네임 변경 완료"} 
                      class:error={saveMessage === "중복된 닉네임입니다. 다시 설정해주세요." || saveMessage === "조건을 충족 해주세요"}>
                      <ion-icon name="checkmark-outline"></ion-icon>
                      {saveMessage}
                    </div>
                  {/if}

                </div>
              </div>
            </div>

            <!--비밀번호 변경 영역-->
            <div id="change-password-container">

              <div id="change-password-Title">
                <t style="color: #bbbbbb;">비밀번호 변경</t>
                {#if ispasswordCheckIcon }
                <ion-icon name="checkmark-outline" style="color: #238636;"></ion-icon>
                {/if}
              </div>

              <!-- 기존 비밀번호 입력 -->
              <div id="change-userPassword_1" class="password-container">

                {#if isOldPasswordVisible}
                <!-- 비밀번호 보이기 -->
                <input 
                  type="text" 
                  placeholder="기존 비밀번호" 
                  bind:value="{inputOldPassword}">
                {:else}
                <!-- 비밀번호 숨기기 -->
                <input 
                  type="password" 
                  placeholder="기존 비밀번호" 
                  bind:value="{inputOldPassword}">
                {/if}

                  <button 
                      type="button" 
                      class="toggle-password-btn" 
                      on:click={toggleOldPasswordVisibility}>
                      <ion-icon name={isOldPasswordVisible ? "eye-off-outline" : "eye-outline"}></ion-icon>
                  </button>
              </div>
          
              <div id="Requirements_password_1" style="color: {oldPasswordStatusColor};" >
                <ion-icon name={oldPasswordIcon}></ion-icon> {oldPasswordStatus || "확인을 위해 비밀번호를 입력해주세요"}
              </div>
          
              <!-- 새 비밀번호 입력 -->
              <div id="change-userPassword_2" class="password-container_2" style="margin-top: 11px;">
                
                {#if isNewPasswordVisible}
                <!-- 비밀번호 보이기 -->
                <input 
                  type="text" 
                  placeholder="새 비밀번호" 
                  bind:value="{inputNewPassword}">
                {:else}
                <!-- 비밀번호 숨기기 -->
                <input 
                  type="password" 
                  placeholder="새 비밀번호" 
                  bind:value="{inputNewPassword}">
                {/if}
                
                  <button 
                    type="button" 
                    class="toggle-password-btn_2" 
                    on:click={toggleNewPasswordVisibility}>
                    <ion-icon name={isNewPasswordVisible ? "eye-off-outline" : "eye-outline"}></ion-icon>
                  </button>
              </div>
          
              <div id="Requirements_password_3" style="color: {condition1Met ? '#238636' : '#9198a1'};">
                <ion-icon name="checkmark-outline"></ion-icon> 영문/숫자/특수문자 중, 2가지 이상 포함
              </div>

              <div id="Requirements_password_4" style="color: {condition2Met ? '#238636' : '#9198a1'};">
                <ion-icon name="checkmark-outline"></ion-icon> 8자 이상 32자 이하 입력 (공백 제외)
              </div>

              <div id="Requirements_password_5" style="color: {condition3Met ? '#238636' : '#9198a1'};">
                <ion-icon name="checkmark-outline"></ion-icon> 연속 3자 이상 동일한 문자/숫자 제외
              </div>
              
              <div id="change-userPassword_3" class="password-container_3" style="margin-top: 11px;">
                
                {#if isNewPasswordVisible_verify}
                <!-- 비밀번호 보이기 -->
                <input 
                  type="text" 
                  placeholder="새 비밀번호 확인" 
                  bind:value="{inputNewPassword_verify}">
                {:else}
                <!-- 비밀번호 숨기기 -->
                <input 
                  type="password" 
                  placeholder="새 비밀번호 확인" 
                  bind:value="{inputNewPassword_verify}">
                {/if}
                
                  <button 
                    type="button" 
                    class="toggle-password-btn_2" 
                    on:click={toggleNewPasswordVisibility_verify}>
                    <ion-icon name={isNewPasswordVisible_verify ? "eye-off-outline" : "eye-outline"}></ion-icon>
                  </button>
              </div>

              <div id="Requirements_password_6" style={verifyPasswordMessageStyle}>
                <ion-icon name="checkmark-outline"></ion-icon> {verifyPasswordMessage}
              </div>

              <div id="password-btn">
                <button id="password-search-Btn">비밀번호 찾기</button>
                <button id="password-save-Btn" on:click={checkOldPassword}>저장</button>
              </div>
            </div> <!--change-password-container 끝-->
          </div> <!--왼쪽 컨테이너 끝-->

        <div id="right-container"> <!--오른쪽 컨테이너 시작-->

          <div id="lawn-container"><!--잔디밭 컨테이너 시작-->
            <img id="lawn" src="https://github.com/user-attachments/assets/5936a6d9-494a-49b6-b9b6-f8333780fd0c">
          </div>

          <div id="community"><!--잔디밭 컨테이너 시작-->
            <img id="log" src="https://github.com/user-attachments/assets/addd25aa-927c-416e-bd39-162d8cff5bf0">
          </div>

        </div ><!--오른쪽 컨테이너 끝-->

      </div> <!--content-box 끝-->
    </div> <!--content 끝-->
  </div> <!--main-container 끝-->
  
  <div class="footer-container">
    <Footer />
  </div>
</main>
  
<style>
  
  main {
    align-items: center;
    transition: transform 0.5s ease, margin 0.5s ease;
    
  }

 .main-container {
   color: white;
   transition: transform 0.5s ease, margin 0.5s ease;
   justify-content: center;
  }
  
 .content {
   column-gap: 35px;
   row-gap: 6px;
  }

  #lawn {
    width: 750px;
    object-fit: cover;
    margin-left: 10px;
    margin-top: 30px;
    position: absolute;
  }

  #log {
    width: 700px;
    object-fit: cover;
    margin-left: 38px;
    margin-top: 0px;
    position: absolute;
  }


 #content-box {
   display: grid;
   grid-template-columns: 0.35fr 0.65fr;
   margin: 0px 0px 0px 200px;
   width: 1200px;
   height: 770px;
   background-color: #151c23;
   border: 3px solid #3d444d;
   border-radius: 8px;
   box-sizing: border-box;
  }

  #left-container {
   display:grid;
   grid-template-rows: 1.05fr 0.95fr;
   border-right: 3px solid #3d444d;
   padding-left: 16px;
   background-color: #12181d;
  }

  #right-container {
    display:grid;
    grid-template-rows: 0.6fr 1fr;
  }

  #profile-edit-container {
    
  }

  #change-userName-container {
    margin-top: 18px;
  }

  #profile-image {
   width: 200px;
   height: 200px;
   border-radius: 50%;
   border: 5px solid #8b8b8b;
   background-color: rgb(88, 88, 88);
   margin: 15px 0px 0px 75px;
  }

  #profile-edit-Btn {
   position: relative;
   background-color: #434343;
   color: white;
   padding: 8px 16px;
   border: 1px solid #888888;
   font-size: 13px;
   font-weight: bold;
   border-radius: 7px;
   cursor: pointer;
   box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3); /* Subtle shadow */
   margin: 150px 0px 0px 150px;
   width: 70px;
  }

  #password-btn {
    margin-top: 17px;
    margin-left: 150px;
  }

  #password-save-Btn {
   position: relative;
   background-color: #3f862d;
   color: white;
   padding: 5px 10px;
   font-size: 14px;
   font-weight: bold;
   border-radius: 7px;
   cursor: pointer;
   box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3); /* Subtle shadow */
   margin: 0px 0px 0px 6px;
   width: 70px;
   height: 30px;
  }

  #password-search-Btn {
   position: relative;
   background-color: #3f862d;
   color: white;
   padding: 5px 10px;
   font-size: 14px;
   font-weight: bold;
   border-radius: 7px;
   cursor: pointer;
   box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3); /* Subtle shadow */
   margin: 0px 0px 0px 6px;
   width: 120px;
   height: 30px;
  }

  #password-save-Btn:hover {
    background-color: #1c682b;
  }

  #password-search-Btn:hover {
    background-color: #1c682b;
  }

  #profile-edit-Btn:hover {
    background-color: #1c682b;
  }

  #change-userName {
    display: flex;
  }

  #change-userName-Title {
    margin: 16px 0px 10px 35px;
    font-weight: bold;
  }

  #change-password-container {
    
  }

  #change-password-Title {
    margin: 0px 0px 10px 35px;
    font-weight: bold;
  }

  #change-userName input[type="text"] {
    padding: 6px;
    margin: 0px 0px 0px 30px;
    border-radius: 7px;
    border: 1px solid #3d444d;
    width: 230px;
    height: 21px;
    background-color: #151b23;
    font-size: 0.9rem;
    color: #FFFFFF;
  }

  .password-container, .password-container_2, .password-container_3 {
    position: relative;
    display: inline-block;
    width: 100%;
  }

  .password-container input {
    padding: 6px;
    margin: 0px 0px 0px 30px;
    border-radius: 7px;
    border: 1px solid #3d444d;
    width: 310px;
    height: 21px;
    background-color: #151b23;
    font-size: 0.9rem;
    color: #FFFFFF;
  }

  .password-container_2 input, .password-container_3 input {
    padding: 6px;
    margin: 0px 0px 0px 30px;
    border-radius: 7px;
    border: 1px solid #3d444d;
    width: 310px;
    height: 21px;
    background-color: #152321;
    font-size: 0.9rem;
    color: #FFFFFF;
  }

  .toggle-password-btn,
  .toggle-password-btn_2 {
      position: absolute;
      top: 57%;
      right: 25px;
      transform: translateY(-50%);
      background: none;
      border: none;
      cursor: pointer;
      font-size: 1.2rem;
      color: #bbbbbb;
      margin-right: 23px;
  }

  .toggle-password-btn:hover {
    color: #424242; /* 버튼 호버 시 색상 변경 */
  }

  #userName-edit-Btn {
   position: relative;
   background-color: #434343;
   color: white;
   border: 1px solid #888888;
   padding: 5px 10px;
   font-size: 14px;
   font-weight: bold;
   border-radius: 7px;
   cursor: pointer;
   box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3); /* Subtle shadow */
   margin: 0px 0px 0px 6px;
   width: 75px;
   height: 34px;
  }

  #Requirements_password_1 {
    margin: 10px 0px 0px 30px;
    font-size: 14px;
    color: #9198a1;
  }

  #Requirements_password_2 {
    margin: 10px 0px 0px 30px;
    font-size: 14px;
    color: #9198a1;
  }

  #Requirements_password_3 {
    margin: 10px 0px 0px 30px;
    font-size: 14px;
    color: #9198a1;
  }

  #Requirements_password_4 {
    margin: 10px 0px 0px 30px;
    font-size: 14px;
    color: #9198a1;
  }

  #Requirements_password_5 {
    margin: 10px 0px 0px 30px;
    font-size: 14px;
    color: #9198a1;
  }

  #Requirements_password_6 {
    margin: 5px 0px 0px 30px;
    font-size: 14px;
    color: #9198a1;
  }

  #userName-edit-Btn:hover {
    background-color: #1c682b;
  }

  #verify-userName-container {
    margin: 15px 0px 0px 30px;
    font-size: 14px;
    color: #9198a1;
  }

  #Requirements_1 {
    margin-bottom: 5px;
  }

  #Requirements_2 {
    margin-bottom: 5px;
  }
    
  #lawn-container {
   columns: 2;
      
  }

  #verify-userName-container div.valid {
    color: #238636; /* 조건을 만족하면 초록색 */
  }

  #verify-userName-container div.invalid {
    color: #9c9c9c; /* 조건을 만족하지 않으면 빨간색 */
  }

  /* 유효한 경우 초록색 */
  .success {
  color: #238636;
  }

  /* 중복된 닉네임인 경우 빨간색 */
  .error {
  color: rgb(173, 44, 44);
  }

</style>
  