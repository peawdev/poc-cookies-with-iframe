<template>
  <div class="app">
    <header class="header">
      <h1>Vue 3 - Google Iframe Integration</h1>
      <p>หน้าบ้านที่เชื่อมต่อกับ Google.com ผ่าน iframe</p>
    </header>

    <main class="main-content">
      <div class="iframe-container">
        <div class="iframe-header">
          <h2>Iframe Content</h2>
          <div class="iframe-url-control">
            <input 
              type="text" 
              v-model="iframeUrl" 
              placeholder="ใส่ URL ของเว็บไซต์"
              class="url-input"
              @keyup.enter="updateIframeUrl"
            >
            <button @click="updateIframeUrl" class="btn btn-primary">โหลด</button>
          </div>
          <div class="controls">
            <button @click="refreshIframe" class="btn btn-primary">รีเฟรช</button>
            <button @click="toggleFullscreen" class="btn btn-secondary">เต็มหน้าจอ</button>
            <button @click="setIframeCookies" class="btn btn-success">Set Cookies</button>
            <button @click="viewIframeCookies" class="btn btn-info">ดู Cookies</button>
            <button @click="sendCookiesToIframe" class="btn btn-warning">ส่ง Cookies</button>
          </div>
        </div>
        
        <div class="iframe-wrapper" :class="{ 'fullscreen': isFullscreen }">
          <iframe
            ref="testIframe"
            :src="iframeUrl"
            title="Iframe Content"
            frameborder="0"
            allowfullscreen
            sandbox="allow-same-origin allow-scripts allow-forms allow-popups allow-popups-to-escape-sandbox"
            @load="onIframeLoad"
          ></iframe>
        </div>
      </div>

      <div class="info-panel">
        <h3>ข้อมูลการเชื่อมต่อ</h3>
        <div class="info-item">
          <strong>สถานะ:</strong> 
          <span :class="connectionStatus.class">{{ connectionStatus.text }}</span>
        </div>
        <div class="info-item">
          <strong>Cookies:</strong> 
          <span>{{ cookiesEnabled ? 'เปิดใช้งาน' : 'ปิดใช้งาน' }}</span>
        </div>
        <div class="info-item">
          <strong>Domain:</strong> {{ currentDomain }}
        </div>
        <div class="info-item">
          <strong>Iframe Domain:</strong> {{ iframeDomain || 'ยังไม่ได้โหลด' }}
        </div>
        <div class="info-item">
          <strong>PostMessage:</strong> 
          <span :class="postMessageStatus.class">{{ postMessageStatus.text }}</span>
        </div>
        <div class="info-item" v-if="sentCookies.length > 0">
          <strong>Cookies ที่ส่ง:</strong> {{ sentCookies.length }} รายการ
        </div>
      </div>

      <!-- Cookie Management Panel -->
      <div class="cookie-panel">
        <h3>จัดการ Cookies</h3>
        
        <!-- Add New Cookie Form -->
        <div class="cookie-form">
          <h4>เพิ่ม Cookie ใหม่</h4>
          <div class="form-group">
            <label for="cookieName">ชื่อ Cookie:</label>
            <input 
              type="text" 
              id="cookieName"
              v-model="newCookie.name" 
              placeholder="เช่น: myCookie"
              class="form-input"
            >
          </div>
          <div class="form-group">
            <label for="cookieValue">ค่า:</label>
            <input 
              type="text" 
              id="cookieValue"
              v-model="newCookie.value" 
              placeholder="เช่น: myValue"
              class="form-input"
            >
          </div>
          <div class="form-group">
            <label for="cookieDomain">Domain:</label>
            <input 
              type="text" 
              id="cookieDomain"
              v-model="newCookie.domain" 
              placeholder="เช่น: .example.com"
              class="form-input"
            >
          </div>
          <div class="form-group">
            <label for="cookiePath">Path:</label>
            <input 
              type="text" 
              id="cookiePath"
              v-model="newCookie.path" 
              placeholder="เช่น: /"
              class="form-input"
            >
          </div>
          <div class="form-group">
            <label for="cookieSameSite">SameSite:</label>
            <select v-model="newCookie.sameSite" class="form-select">
              <option value="Lax">Lax</option>
              <option value="Strict">Strict</option>
              <option value="None">None</option>
            </select>
          </div>
          <div class="form-group">
            <label class="checkbox-label">
              <input type="checkbox" v-model="newCookie.secure">
              Secure (HTTPS only)
            </label>
          </div>
          <div class="form-group">
            <label for="cookieExpires">วันหมดอายุ (วัน):</label>
            <input 
              type="number" 
              id="cookieExpires"
              v-model="newCookie.expires" 
              placeholder="เช่น: 7"
              class="form-input"
            >
          </div>
          <button @click="addCookie" class="btn btn-success">เพิ่ม Cookie</button>
        </div>

        <!-- Current Cookies List -->
        <div class="cookies-list">
          <h4>Cookies ปัจจุบัน</h4>
          <div v-if="currentCookies.length === 0" class="no-cookies">
            ไม่มี cookies
          </div>
          <div v-else class="cookie-item" v-for="(cookie, index) in currentCookies" :key="index">
            <div class="cookie-info">
              <strong>{{ cookie.name }}</strong> = {{ cookie.value }}
              <div class="cookie-details">
                Domain: {{ cookie.domain || 'current' }} | 
                Path: {{ cookie.path || '/' }} | 
                SameSite: {{ cookie.sameSite || 'Lax' }}
                <span v-if="cookie.secure"> | Secure</span>
              </div>
            </div>
            <button @click="deleteCookie(cookie.name)" class="btn btn-danger btn-sm">ลบ</button>
          </div>
        </div>

        <!-- Iframe Cookies List -->
        <div class="iframe-cookies-list" v-if="iframeCookies.length > 0">
          <h4>Iframe Cookies ({{ iframeDomain }})</h4>
          <div class="cookie-item" v-for="(cookie, index) in iframeCookies" :key="'iframe-' + index">
            <div class="cookie-info">
              <strong>{{ cookie.name }}</strong> = {{ cookie.value }}
              <div class="cookie-details">
                Domain: {{ cookie.domain }} | 
                Path: {{ cookie.path }} | 
                SameSite: {{ cookie.sameSite }}
                <span v-if="cookie.secure"> | Secure</span>
              </div>
            </div>
            <button @click="deleteCookie(cookie.name)" class="btn btn-danger btn-sm">ลบ</button>
          </div>
        </div>
      </div>
    </main>

    <!-- Iframe Cookies Modal -->
    <div v-if="showIframeCookiesModal" class="modal-overlay" @click="closeIframeCookiesModal">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h3>Cookies ของ Iframe ({{ iframeDomain }})</h3>
          <button @click="closeIframeCookiesModal" class="btn-close">&times;</button>
        </div>
        <div class="modal-body">
          <div v-if="iframeCookiesFromIframe.length === 0" class="no-cookies">
            ไม่พบ cookies ใน iframe
          </div>
          <div v-else class="cookies-table">
            <table>
              <thead>
                <tr>
                  <th>ชื่อ</th>
                  <th>ค่า</th>
                  <th>Domain</th>
                  <th>Path</th>
                  <th>SameSite</th>
                  <th>Secure</th>
                  <th>Expires</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(cookie, index) in iframeCookiesFromIframe" :key="index">
                  <td>{{ cookie.name }}</td>
                  <td class="cookie-value">{{ cookie.value }}</td>
                  <td>{{ cookie.domain || '-' }}</td>
                  <td>{{ cookie.path || '/' }}</td>
                  <td>{{ cookie.sameSite || 'Lax' }}</td>
                  <td>{{ cookie.secure ? 'Yes' : 'No' }}</td>
                  <td>{{ cookie.expires || 'Session' }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
        <div class="modal-footer">
          <button @click="refreshIframeCookies" class="btn btn-primary">รีเฟรช</button>
          <button @click="closeIframeCookiesModal" class="btn btn-secondary">ปิด</button>
        </div>
      </div>
    </div>

    <footer class="footer">
      <p>&copy; 2024 Vue 3 Iframe Project</p>
    </footer>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted } from 'vue'

export default {
  name: 'App',
  setup() {
    const testIframe = ref(null)
    const isFullscreen = ref(false)
    const connectionStatus = ref({ text: 'กำลังโหลด...', class: 'loading' })
    const cookiesEnabled = ref(false)
    const currentDomain = ref('')
    
    // Iframe and Cookie management
    const iframeUrl = ref('http://localhost:3000/iframe-receiver.html')
    const iframeDomain = ref('')
    const newCookie = ref({
      name: '',
      value: '',
      domain: '',
      path: '/',
      sameSite: 'Lax',
      secure: false,
      expires: 7
    })
    const currentCookies = ref([])
    const iframeCookies = ref([])
    const showIframeCookiesModal = ref(false)
    const iframeCookiesFromIframe = ref([])
    const postMessageStatus = ref({ text: 'พร้อมส่ง', class: 'success' })
    const sentCookies = ref([])

    // ตั้งค่า cookies สำหรับ cross-domain
    const setupCrossDomainCookies = () => {
      try {
        document.cookie = "vueApp=true; domain=localhost; path=/; SameSite=Lax"
        document.cookie = "localApp=true; path=/; SameSite=Lax"
        
        cookiesEnabled.value = true
        console.log('Cross-domain cookies setup completed')
      } catch (error) {
        console.error('Error setting up cookies:', error)
        cookiesEnabled.value = false
      }
    }

    // จัดการเมื่อ iframe โหลดเสร็จ
    const onIframeLoad = () => {
      connectionStatus.value = { text: 'เชื่อมต่อสำเร็จ', class: 'success' }
      extractIframeDomain()
      console.log('Iframe loaded successfully:', iframeUrl.value)
    }

    // แยก domain จาก URL
    const extractIframeDomain = () => {
      try {
        const url = new URL(iframeUrl.value)
        iframeDomain.value = url.hostname
        console.log('Iframe domain:', iframeDomain.value)
      } catch (error) {
        console.error('Error extracting domain:', error)
        iframeDomain.value = ''
      }
    }

    // อัปเดต iframe URL
    const updateIframeUrl = () => {
      if (iframeUrl.value) {
        connectionStatus.value = { text: 'กำลังโหลด...', class: 'loading' }
        extractIframeDomain()
        console.log('Updating iframe URL to:', iframeUrl.value)
      }
    }

    // รีเฟรช iframe
    const refreshIframe = () => {
      if (testIframe.value) {
        connectionStatus.value = { text: 'กำลังรีเฟรช...', class: 'loading' }
        testIframe.value.src = testIframe.value.src
      }
    }

    // เปิด/ปิดเต็มหน้าจอ
    const toggleFullscreen = () => {
      isFullscreen.value = !isFullscreen.value
    }

    // ตั้งค่า domain ปัจจุบัน
    const setCurrentDomain = () => {
      currentDomain.value = window.location.hostname
    }

    // Cookie management functions
    const addCookie = () => {
      if (!newCookie.value.name || !newCookie.value.value) {
        alert('กรุณากรอกชื่อและค่าของ cookie')
        return
      }

      try {
        let cookieString = `${newCookie.value.name}=${newCookie.value.value}`
        
        if (newCookie.value.domain) {
          cookieString += `; domain=${newCookie.value.domain}`
        }
        
        if (newCookie.value.path) {
          cookieString += `; path=${newCookie.value.path}`
        }
        
        if (newCookie.value.sameSite) {
          cookieString += `; SameSite=${newCookie.value.sameSite}`
        }
        
        if (newCookie.value.secure) {
          cookieString += `; Secure`
        }
        
        if (newCookie.value.expires) {
          const date = new Date()
          date.setTime(date.getTime() + (newCookie.value.expires * 24 * 60 * 60 * 1000))
          cookieString += `; expires=${date.toUTCString()}`
        }

        document.cookie = cookieString
        loadCurrentCookies()
        
        // Reset form
        newCookie.value = {
          name: '',
          value: '',
          domain: '',
          path: '/',
          sameSite: 'Lax',
          secure: false,
          expires: 7
        }
        
        console.log('Cookie added:', cookieString)
      } catch (error) {
        console.error('Error adding cookie:', error)
        alert('เกิดข้อผิดพลาดในการเพิ่ม cookie')
      }
    }

    const deleteCookie = (cookieName) => {
      try {
        document.cookie = `${cookieName}=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/`
        loadCurrentCookies()
        console.log('Cookie deleted:', cookieName)
      } catch (error) {
        console.error('Error deleting cookie:', error)
        alert('เกิดข้อผิดพลาดในการลบ cookie')
      }
    }

    const loadCurrentCookies = () => {
      try {
        const cookies = document.cookie.split(';')
        const cookieList = []
        
        cookies.forEach(cookie => {
          const trimmedCookie = cookie.trim()
          if (trimmedCookie) {
            const [name, value] = trimmedCookie.split('=')
            if (name && value) {
              cookieList.push({
                name: name.trim(),
                value: value.trim(),
                domain: 'current',
                path: '/',
                sameSite: 'Lax',
                secure: false
              })
            }
          }
        })
        
        currentCookies.value = cookieList
      } catch (error) {
        console.error('Error loading cookies:', error)
        currentCookies.value = []
      }
    }

    // Set cookies สำหรับ iframe domain
    const setIframeCookies = () => {
      if (!iframeDomain.value) {
        alert('ไม่สามารถระบุ domain ของ iframe ได้')
        return
      }

      try {
        // Cookies สำหรับ iframe domain
        const iframeCookiesToSet = [
          {
            name: 'sessionId',
            value: 'iframe-session-' + Date.now(),
            domain: iframeDomain.value,
            path: '/',
            sameSite: 'None',
            secure: true,
            expires: 1
          },
          {
            name: 'userToken',
            value: 'token-' + Math.random().toString(36).substr(2, 9),
            domain: iframeDomain.value,
            path: '/',
            sameSite: 'None',
            secure: true,
            expires: 7
          },
          {
            name: 'iframeAuth',
            value: 'authenticated',
            domain: iframeDomain.value,
            path: '/',
            sameSite: 'None',
            secure: true,
            expires: 30
          }
        ]

        // Set cookies สำหรับ iframe domain
        iframeCookiesToSet.forEach(cookie => {
          let cookieString = `${cookie.name}=${cookie.value}`
          
          if (cookie.domain) {
            cookieString += `; domain=${cookie.domain}`
          }
          
          if (cookie.path) {
            cookieString += `; path=${cookie.path}`
          }
          
          if (cookie.sameSite) {
            cookieString += `; SameSite=${cookie.sameSite}`
          }
          
          if (cookie.secure) {
            cookieString += `; Secure`
          }
          
          if (cookie.expires) {
            const date = new Date()
            date.setTime(date.getTime() + (cookie.expires * 24 * 60 * 60 * 1000))
            cookieString += `; expires=${date.toUTCString()}`
          }

          document.cookie = cookieString
          console.log('Iframe cookie set:', cookieString)
        })

        // อัปเดตรายการ cookies
        loadCurrentCookies()
        loadIframeCookies()
        
        alert(`ตั้งค่า cookies สำหรับ domain ${iframeDomain.value} เรียบร้อยแล้ว`)
      } catch (error) {
        console.error('Error setting iframe cookies:', error)
        alert('เกิดข้อผิดพลาดในการตั้งค่า cookies สำหรับ iframe')
      }
    }

    // โหลด cookies ที่เกี่ยวข้องกับ iframe
    const loadIframeCookies = () => {
      try {
        const cookies = document.cookie.split(';')
        const iframeCookieList = []
        
        cookies.forEach(cookie => {
          const trimmedCookie = cookie.trim()
          if (trimmedCookie) {
            const [name, value] = trimmedCookie.split('=')
            if (name && value && iframeDomain.value) {
              // ตรวจสอบว่า cookie นี้เกี่ยวข้องกับ iframe domain หรือไม่
              if (name.includes('sessionId') || name.includes('userToken') || name.includes('iframeAuth')) {
                iframeCookieList.push({
                  name: name.trim(),
                  value: value.trim(),
                  domain: iframeDomain.value,
                  path: '/',
                  sameSite: 'None',
                  secure: true
                })
              }
            }
          }
        })
        
        iframeCookies.value = iframeCookieList
      } catch (error) {
        console.error('Error loading iframe cookies:', error)
        iframeCookies.value = []
      }
    }

    // ดู cookies ของ iframe
    const viewIframeCookies = () => {
      try {
        if (!testIframe.value) {
          alert('ไม่พบ iframe')
          return
        }

        // พยายามเข้าถึง cookies ของ iframe
        try {
          const iframeDocument = testIframe.value.contentDocument || testIframe.value.contentWindow.document
          const iframeCookies = iframeDocument.cookie
          
          if (iframeCookies) {
            parseIframeCookies(iframeCookies)
          } else {
            // ถ้าไม่สามารถเข้าถึงได้ ให้ใช้วิธีอื่น
            getIframeCookiesViaPostMessage()
          }
        } catch (error) {
          console.log('Cannot access iframe cookies directly, trying postMessage method')
          getIframeCookiesViaPostMessage()
        }

        showIframeCookiesModal.value = true
      } catch (error) {
        console.error('Error viewing iframe cookies:', error)
        alert('ไม่สามารถดู cookies ของ iframe ได้ เนื่องจากข้อจำกัดของ Same-Origin Policy')
        showIframeCookiesModal.value = true
      }
    }

    // แปลง cookies string เป็น array
    const parseIframeCookies = (cookieString) => {
      try {
        const cookies = cookieString.split(';')
        const cookieList = []
        
        cookies.forEach(cookie => {
          const trimmedCookie = cookie.trim()
          if (trimmedCookie) {
            const [name, value] = trimmedCookie.split('=')
            if (name && value) {
              cookieList.push({
                name: name.trim(),
                value: value.trim(),
                domain: iframeDomain.value,
                path: '/',
                sameSite: 'Unknown',
                secure: false,
                expires: 'Session'
              })
            }
          }
        })
        
        iframeCookiesFromIframe.value = cookieList
      } catch (error) {
        console.error('Error parsing iframe cookies:', error)
        iframeCookiesFromIframe.value = []
      }
    }

    // ใช้ postMessage เพื่อขอ cookies จาก iframe
    const getIframeCookiesViaPostMessage = () => {
      try {
        // ส่งข้อความไปยัง iframe เพื่อขอ cookies
        testIframe.value.contentWindow.postMessage({
          type: 'GET_COOKIES_REQUEST',
          source: 'parent'
        }, '*')

        // ฟังข้อความตอบกลับ
        const handleMessage = (event) => {
          if (event.data.type === 'GET_COOKIES_RESPONSE') {
            parseIframeCookies(event.data.cookies || '')
            window.removeEventListener('message', handleMessage)
          }
        }

        window.addEventListener('message', handleMessage)
        
        // ถ้าไม่ได้รับคำตอบใน 3 วินาที ให้แสดงข้อความว่าไม่สามารถเข้าถึงได้
        setTimeout(() => {
          window.removeEventListener('message', handleMessage)
          if (iframeCookiesFromIframe.value.length === 0) {
            iframeCookiesFromIframe.value = [{
              name: 'Access Denied',
              value: 'Cannot access iframe cookies due to Same-Origin Policy',
              domain: iframeDomain.value,
              path: '/',
              sameSite: 'Unknown',
              secure: false,
              expires: 'N/A'
            }]
          }
        }, 3000)
      } catch (error) {
        console.error('Error getting cookies via postMessage:', error)
        iframeCookiesFromIframe.value = [{
          name: 'Error',
          value: 'Failed to access iframe cookies',
          domain: iframeDomain.value,
          path: '/',
          sameSite: 'Unknown',
          secure: false,
          expires: 'N/A'
        }]
      }
    }

    // ปิด modal
    const closeIframeCookiesModal = () => {
      showIframeCookiesModal.value = false
      iframeCookiesFromIframe.value = []
    }

    // รีเฟรช cookies ของ iframe
    const refreshIframeCookies = () => {
      viewIframeCookies()
    }

    // ส่ง cookies ไปยัง iframe ผ่าน postMessage
    const sendCookiesToIframe = () => {
      try {
        if (!testIframe.value) {
          alert('ไม่พบ iframe')
          return
        }

        if (!iframeDomain.value) {
          alert('ไม่สามารถระบุ domain ของ iframe ได้')
          return
        }

        // เตรียม cookies ที่จะส่ง
        const cookiesToSend = prepareCookiesForIframe()
        
        // ส่ง cookies ผ่าน postMessage
        const message = {
          type: 'SEND_COOKIES',
          source: 'parent',
          cookies: cookiesToSend,
          timestamp: Date.now()
        }

        testIframe.value.contentWindow.postMessage(message, '*')
        
        postMessageStatus.value = { text: 'กำลังส่ง...', class: 'loading' }
        sentCookies.value = cookiesToSend
        
        console.log('Sending cookies to iframe:', message)
        
        // รีเซ็ตสถานะหลังจาก 3 วินาที
        setTimeout(() => {
          postMessageStatus.value = { text: 'ส่งเรียบร้อย', class: 'success' }
        }, 3000)
        
      } catch (error) {
        console.error('Error sending cookies to iframe:', error)
        postMessageStatus.value = { text: 'ส่งล้มเหลว', class: 'error' }
        alert('เกิดข้อผิดพลาดในการส่ง cookies ไปยัง iframe')
      }
    }

    // เตรียม cookies สำหรับส่งไปยัง iframe
    const prepareCookiesForIframe = () => {
      try {
        const cookies = document.cookie.split(';')
        const cookiesToSend = []
        
        cookies.forEach(cookie => {
          const trimmedCookie = cookie.trim()
          if (trimmedCookie) {
            const [name, value] = trimmedCookie.split('=')
            if (name && value) {
              cookiesToSend.push({
                name: name.trim(),
                value: value.trim(),
                domain: currentDomain.value,
                path: '/',
                sameSite: 'Lax',
                secure: false,
                expires: 'Session'
              })
            }
          }
        })
        
        return cookiesToSend
      } catch (error) {
        console.error('Error preparing cookies:', error)
        return []
      }
    }

    // ฟัง message จาก iframe
    const handleIframeMessage = (event) => {
      try {
        // ตรวจสอบ origin ของ message
        if (event.origin !== window.location.origin && !event.origin.includes('thesonicblue.xyz')) {
          console.log('Message from untrusted origin:', event.origin)
          return
        }

        console.log('Received message from iframe:', event.data)

        switch (event.data.type) {
          case 'COOKIES_RECEIVED':
            postMessageStatus.value = { text: 'iframe ได้รับ cookies', class: 'success' }
            console.log('Iframe confirmed receipt of cookies')
            break
            
          case 'COOKIES_ERROR':
            postMessageStatus.value = { text: 'iframe ไม่สามารถรับ cookies', class: 'error' }
            console.error('Iframe reported error:', event.data.error)
            break
            
          case 'REQUEST_COOKIES':
            // ส่ง cookies กลับไปยัง iframe เมื่อ iframe ขอ
            const cookiesToSend = prepareCookiesForIframe()
            event.source.postMessage({
              type: 'COOKIES_RESPONSE',
              source: 'parent',
              cookies: cookiesToSend,
              timestamp: Date.now()
            }, event.origin)
            console.log('Sent cookies in response to request')
            break
            
          case 'GET_COOKIES_RESPONSE':
            // รับ cookies จาก iframe (สำหรับฟังก์ชันดู cookies)
            if (event.data.cookies) {
              parseIframeCookies(event.data.cookies)
            }
            break
            
          default:
            console.log('Unknown message type:', event.data.type)
        }
      } catch (error) {
        console.error('Error handling iframe message:', error)
      }
    }

    // จัดการ keyboard shortcuts
    const handleKeydown = (event) => {
      if (event.key === 'F11') {
        event.preventDefault()
        toggleFullscreen()
      }
    }

    onMounted(() => {
      setupCrossDomainCookies()
      setCurrentDomain()
      loadCurrentCookies()
      extractIframeDomain()
      window.addEventListener('keydown', handleKeydown)
      window.addEventListener('message', handleIframeMessage)
      
      // ตรวจสอบการเชื่อมต่อ
      setTimeout(() => {
        if (connectionStatus.value.text === 'กำลังโหลด...') {
          connectionStatus.value = { text: 'เชื่อมต่อล้มเหลว', class: 'error' }
        }
      }, 10000)
    })

    onUnmounted(() => {
      window.removeEventListener('keydown', handleKeydown)
      window.removeEventListener('message', handleIframeMessage)
    })

    return {
      testIframe,
      isFullscreen,
      connectionStatus,
      cookiesEnabled,
      currentDomain,
      iframeUrl,
      iframeDomain,
      newCookie,
      currentCookies,
      iframeCookies,
      showIframeCookiesModal,
      iframeCookiesFromIframe,
      postMessageStatus,
      sentCookies,
      onIframeLoad,
      refreshIframe,
      toggleFullscreen,
      updateIframeUrl,
      addCookie,
      deleteCookie,
      loadCurrentCookies,
      setIframeCookies,
      loadIframeCookies,
      viewIframeCookies,
      closeIframeCookiesModal,
      refreshIframeCookies,
      sendCookiesToIframe,
      prepareCookiesForIframe,
      handleIframeMessage
    }
  }
}
</script>

<style scoped>
.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 2rem;
  text-align: center;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.header h1 {
  margin: 0 0 0.5rem 0;
  font-size: 2.5rem;
  font-weight: 700;
}

.header p {
  margin: 0;
  font-size: 1.1rem;
  opacity: 0.9;
}

.main-content {
  flex: 1;
  padding: 2rem;
  display: grid;
  grid-template-columns: 2fr 1fr 1fr;
  gap: 2rem;
  max-width: 1600px;
  margin: 0 auto;
}

.iframe-container {
  background: white;
  border-radius: 12px;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.iframe-header {
  background: #f8f9fa;
  padding: 1rem 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
  border-bottom: 1px solid #e9ecef;
}

.iframe-url-control {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.url-input {
  flex: 1;
  padding: 0.5rem;
  border: 1px solid #ced4da;
  border-radius: 4px;
  font-size: 0.9rem;
}

.url-input:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.iframe-header-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.iframe-header h2 {
  margin: 0;
  color: #495057;
  font-size: 1.3rem;
}

.iframe-cookies-list {
  margin-top: 1.5rem;
  padding-top: 1.5rem;
  border-top: 1px solid #e9ecef;
}

.iframe-cookies-list h4 {
  margin: 0 0 1rem 0;
  color: #495057;
  font-size: 1rem;
  background: #e3f2fd;
  padding: 0.5rem;
  border-radius: 4px;
}

.controls {
  display: flex;
  gap: 0.5rem;
}

.btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.2s ease;
}

.btn-primary {
  background: #007bff;
  color: white;
}

.btn-primary:hover {
  background: #0056b3;
  transform: translateY(-1px);
}

.btn-secondary {
  background: #6c757d;
  color: white;
}

.btn-secondary:hover {
  background: #545b62;
  transform: translateY(-1px);
}

.btn-success {
  background: #28a745;
  color: white;
}

.btn-success:hover {
  background: #1e7e34;
  transform: translateY(-1px);
}

.btn-danger {
  background: #dc3545;
  color: white;
}

.btn-danger:hover {
  background: #c82333;
  transform: translateY(-1px);
}

.btn-sm {
  padding: 0.25rem 0.5rem;
  font-size: 0.875rem;
}

.btn-info {
  background: #17a2b8;
  color: white;
}

.btn-info:hover {
  background: #138496;
  transform: translateY(-1px);
}

.btn-warning {
  background: #ffc107;
  color: #212529;
}

.btn-warning:hover {
  background: #e0a800;
  transform: translateY(-1px);
}

.btn-close {
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  color: #6c757d;
  padding: 0;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.btn-close:hover {
  color: #495057;
}

.iframe-wrapper {
  position: relative;
  height: 600px;
  transition: all 0.3s ease;
}

.iframe-wrapper.fullscreen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 9999;
  background: white;
}

.iframe-wrapper iframe {
  width: 100%;
  height: 100%;
  border: none;
}

.info-panel {
  background: white;
  padding: 1.5rem;
  border-radius: 12px;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
  height: fit-content;
}

.info-panel h3 {
  margin: 0 0 1rem 0;
  color: #495057;
  font-size: 1.2rem;
}

.info-item {
  margin-bottom: 0.75rem;
  padding: 0.5rem 0;
  border-bottom: 1px solid #f1f3f4;
}

.info-item:last-child {
  border-bottom: none;
}

.info-item strong {
  color: #495057;
  margin-right: 0.5rem;
}

.loading {
  color: #ffc107;
  font-weight: 500;
}

.success {
  color: #28a745;
  font-weight: 500;
}

.error {
  color: #dc3545;
  font-weight: 500;
}

/* Cookie Panel Styles */
.cookie-panel {
  background: white;
  padding: 1.5rem;
  border-radius: 12px;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
  height: fit-content;
}

.cookie-panel h3 {
  margin: 0 0 1.5rem 0;
  color: #495057;
  font-size: 1.2rem;
}

.cookie-form {
  margin-bottom: 2rem;
  padding-bottom: 1.5rem;
  border-bottom: 1px solid #e9ecef;
}

.cookie-form h4 {
  margin: 0 0 1rem 0;
  color: #495057;
  font-size: 1rem;
}

.form-group {
  margin-bottom: 1rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  color: #495057;
  font-weight: 500;
  font-size: 0.9rem;
}

.form-input, .form-select {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ced4da;
  border-radius: 4px;
  font-size: 0.9rem;
  transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}

.form-input:focus, .form-select:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.checkbox-label {
  display: flex;
  align-items: center;
  cursor: pointer;
}

.checkbox-label input[type="checkbox"] {
  margin-right: 0.5rem;
}

.cookies-list h4 {
  margin: 0 0 1rem 0;
  color: #495057;
  font-size: 1rem;
}

.no-cookies {
  color: #6c757d;
  font-style: italic;
  text-align: center;
  padding: 1rem;
}

.cookie-item {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 0.75rem;
  margin-bottom: 0.5rem;
  background: #f8f9fa;
  border-radius: 6px;
  border: 1px solid #e9ecef;
}

.cookie-info {
  flex: 1;
  margin-right: 0.5rem;
}

.cookie-info strong {
  color: #495057;
  font-size: 0.9rem;
}

.cookie-details {
  font-size: 0.8rem;
  color: #6c757d;
  margin-top: 0.25rem;
}

.footer {
  background: #343a40;
  color: white;
  text-align: center;
  padding: 1rem;
  margin-top: auto;
}

.footer p {
  margin: 0;
  opacity: 0.8;
}

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10000;
}

.modal-content {
  background: white;
  border-radius: 12px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
  max-width: 90vw;
  max-height: 90vh;
  width: 800px;
  display: flex;
  flex-direction: column;
}

.modal-header {
  padding: 1.5rem;
  border-bottom: 1px solid #e9ecef;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.modal-header h3 {
  margin: 0;
  color: #495057;
  font-size: 1.3rem;
}

.modal-body {
  padding: 1.5rem;
  overflow-y: auto;
  flex: 1;
}

.modal-footer {
  padding: 1rem 1.5rem;
  border-top: 1px solid #e9ecef;
  display: flex;
  justify-content: flex-end;
  gap: 0.5rem;
}

.cookies-table {
  overflow-x: auto;
}

.cookies-table table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9rem;
}

.cookies-table th,
.cookies-table td {
  padding: 0.75rem;
  text-align: left;
  border-bottom: 1px solid #e9ecef;
}

.cookies-table th {
  background: #f8f9fa;
  font-weight: 600;
  color: #495057;
  position: sticky;
  top: 0;
}

.cookies-table tr:hover {
  background: #f8f9fa;
}

.cookie-value {
  max-width: 200px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

@media (max-width: 1200px) {
  .main-content {
    grid-template-columns: 1fr 1fr;
  }
}

@media (max-width: 768px) {
  .main-content {
    grid-template-columns: 1fr;
    padding: 1rem;
  }
  
  .header h1 {
    font-size: 2rem;
  }
  
  .iframe-wrapper {
    height: 400px;
  }
  
  .controls {
    flex-direction: column;
  }
  
  .cookie-item {
    flex-direction: column;
    align-items: stretch;
  }
  
  .cookie-info {
    margin-right: 0;
    margin-bottom: 0.5rem;
  }
  
  .modal-content {
    width: 95vw;
    max-height: 95vh;
  }
  
  .cookies-table {
    font-size: 0.8rem;
  }
  
  .cookies-table th,
  .cookies-table td {
    padding: 0.5rem;
  }
}
</style>
