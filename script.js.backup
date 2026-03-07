// Pulse Underground Core Functionality
class PulseUnderground {
    constructor() {
        this.wallet = null;
        this.connectedAddress = null;
        this.currentChannel = 'main';
        this.messages = [];
        this.polling = false;
        this.init();
    }

    async init() {
        this.setupEventListeners();
        this.checkWalletConnection();
        this.loadMessages();
    }

    setupEventListeners() {
        // Button interactions
        document.querySelectorAll('.btn').forEach(btn => {
            btn.addEventListener('click', (e) => this.handleButtonClick(e));
            btn.addEventListener('mouseenter', (e) => this.handleButtonHover(e));
            btn.addEventListener('mouseleave', (e) => this.handleButtonLeave(e));
        });

        // Channel switching
        document.querySelectorAll('.channel-btn').forEach(btn => {
            btn.addEventListener('click', (e) => this.switchChannel(e.target.dataset.channel));
        });

        // Post message
        const postBtn = document.getElementById('post-btn');
        if (postBtn) {
            postBtn.addEventListener('click', () => this.postMessage());
        }

        // Search
        const searchBtn = document.getElementById('search-btn');
        if (searchBtn) {
            searchBtn.addEventListener('click', () => this.searchMessages());
        }

        // Infinite scroll
        window.addEventListener('scroll', () => this.handleScroll());
    }

    handleButtonClick(e) {
        const btn = e.currentTarget;
        btn.classList.add('active');
        setTimeout(() => btn.classList.remove('active'), 200);
    }

    handleButtonHover(e) {
        const btn = e.currentTarget;
        btn.style.transform = 'translateY(-2px)';
        btn.style.boxShadow = '0 4px 20px rgba(124, 77, 255, 0.4)';
    }

    handleButtonLeave(e) {
        const btn = e.currentTarget;
        btn.style.transform = 'translateY(0)';
        btn.style.boxShadow = '0 2px 10px rgba(124, 77, 255, 0.2)';
    }

    async checkWalletConnection() {
        try {
            if (typeof window.ethereum !== 'undefined') {
                const accounts = await window.ethereum.request({ method: 'eth_requestAccounts' });
                this.wallet = window.ethereum;
                this.connectedAddress = accounts[0];
                this.updateConnectionStatus('Connected', 'success');
                this.startPolling();
            } else {
                this.updateConnectionStatus('Connect Wallet', 'default');
            }
        } catch (error) {
            console.error('Wallet connection error:', error);
            this.updateConnectionStatus('Error', 'error');
        }
    }

    async postMessage() {
        const messageInput = document.getElementById('message-input');
        const message = messageInput.value.trim();
        
        if (!message) {
            this.showNotification('Message cannot be empty', 'error');
            return;
        }

        if (!this.connectedAddress) {
            this.showNotification('Please connect wallet first', 'error');
            return;
        }

        try {
            this.setButtonLoading('post-btn', true);
            
            // Create transaction
            const tx = await this.wallet.request({
                method: 'eth_sendTransaction',
                params: [{
                    from: this.connectedAddress,
                    to: this.currentChannel === 'main' ? '0xMainChannel' : this.connectedAddress,
                    data: '0x' + Buffer.from(message).toString('hex'),
                    value: '0x0',
                    gas: '0x' + (21000 + message.length * 20).toString(16)
                }]
            });

            this.showNotification('Message sent!', 'success');
            messageInput.value = '';
            this.loadMessages();
        } catch (error) {
            console.error('Transaction error:', error);
            this.showNotification('Failed to send message', 'error');
        } finally {
            this.setButtonLoading('post-btn', false);
        }
    }

    async loadMessages() {
        try {
            const response = await fetch(`https://api.pulsescan.io/v2/transactions/address/${this.currentChannel}`);
            const data = await response.json();
            
            // Filter and decode messages
            this.messages = data.transactions
                .filter(tx => tx.to === this.currentChannel)
                .map(tx => ({
                    id: tx.hash,
                    from: tx.from,
                    content: Buffer.from(tx.input.substring(2), 'hex').toString(),
                    timestamp: new Date(tx.timeStamp * 1000)
                }));
            
            this.renderMessages();
        } catch (error) {
            console.error('Failed to load messages:', error);
        }
    }

    async searchMessages() {
        const searchInput = document.getElementById('search-input');
        const query = searchInput.value.trim().toLowerCase();
        
        if (!query) return;

        const filtered = this.messages.filter(msg => 
            msg.content.toLowerCase().includes(query) || 
            msg.from.toLowerCase().includes(query)
        );
        
        this.renderMessages(filtered);
    }

    switchChannel(channel) {
        this.currentChannel = channel;
        document.querySelectorAll('.channel-btn').forEach(btn => {
            btn.classList.toggle('active', btn.dataset.channel === channel);
        });
        this.loadMessages();
        this.showNotification(`Switched to ${channel} channel`, 'info');
    }

    startPolling() {
        if (this.polling) return;
        this.polling = true;
        this.pollInterval = setInterval(() => this.loadMessages(), 30000);
    }

    stopPolling() {
        if (this.polling) {
            clearInterval(this.pollInterval);
            this.polling = false;
        }
    }

    renderMessages(messages = this.messages) {
        const container = document.getElementById('messages-container');
        if (!container) return;

        if (messages.length === 0) {
            container.innerHTML = '<div class="no-messages">No messages yet</div>';
            return;
        }

        container.innerHTML = messages.map(msg => `
            <div class="message">
                <div class="message-header">
                    <span class="message-address">${msg.from.substring(0, 8)}...</span>
                    <span class="message-time">${this.formatTime(msg.timestamp)}</span>
                </div>
                <div class="message-content">${this.escapeHtml(msg.content)}</div>
            </div>
        `).join('');
    }

    formatTime(date) {
        return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    }

    escapeHtml(text) {
        const map = {
            '&': '&amp;',
            '<': '&lt;',
            '>': '&gt;',
            '"': '&quot;',
            "'": '&#039;'
        };
        return text.replace(/[&<>"']/g, m => map[m]);
    }

    updateConnectionStatus(status, type) {
        const statusEl = document.getElementById('connection-status');
        if (statusEl) {
            statusEl.textContent = status;
            statusEl.className = `connection-status ${type}`;
        }
    }

    setButtonLoading(btnId, loading) {
        const btn = document.getElementById(btnId);
        if (btn) {
            if (loading) {
                btn.disabled = true;
                btn.innerHTML = '<span class="loading"></span> Sending...';
            } else {
                btn.disabled = false;
                btn.innerHTML = 'Send';
            }
        }
    }

    showNotification(message, type) {
        // Create notification
        const notification = document.createElement('div');
        notification.className = `notification notification-${type}`;
        notification.textContent = message;
        notification.style.cssText = `
            position: fixed;
            top: 20px;
            right: 20px;
            padding: 12px 20px;
            background: ${type === 'success' ? '#4CAF50' : type === 'error' ? '#f44336' : '#2196F3'};
            color: white;
            borderRadius: 8px;
            boxShadow: 0 4px 20px rgba(0,0,0,0.2);
            zIndex: 1000;
            animation: slideIn 0.3s ease-out;
        `;
        
        // Add animation
        const style = document.createElement('style');
        style.textContent = `
            @keyframes slideIn {
                from { transform: translateX(100%); opacity: 0; }
                to { transform: translateX(0); opacity: 1; }
            }
            @keyframes slideOut {
                from { transform: translateX(0); opacity: 1; }
                to { transform: translateX(100%); opacity: 0; }
            }
        `;
        document.head.appendChild(style);
        
        document.body.appendChild(notification);
        
        // Remove after 3 seconds
        setTimeout(() => {
            notification.style.animation = 'slideOut 0.3s ease-out';
            setTimeout(() => notification.remove(), 300);
        }, 3000);
    }

    handleScroll() {
        const container = document.getElementById('messages-container');
        if (!container) return;

        const { scrollTop, scrollHeight, clientHeight } = container;
        if (scrollTop + clientHeight >= scrollHeight - 100) {
            // Load more messages
            this.loadMoreMessages();
        }
    }

    async loadMoreMessages() {
        // Implement infinite scroll loading
        console.log('Loading more messages...');
    }
}

// Initialize when DOM is ready
document.addEventListener('DOMContentLoaded', () => {
    window.pulseUnderground = new PulseUnderground();
});

// Export for external use
window.PulseUnderground = PulseUnderground;
