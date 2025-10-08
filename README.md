# e-cot
E-Cot is an innovative, technology-driven smart cot designed to provide comfort, safety, and convenience through intelligent automation. It integrates modern electronics with ergonomic design to create a next-generation sleeping and caregiving solution.
import React, { useState, useMemo, useEffect } from 'react';
import { ShoppingCart, X, Plus, Minus, ArrowLeft, CheckCircle, MapPin, Tag, Search, Filter, Home, Info, Phone, Heart, Globe, Mail } from 'lucide-react';

// --- Global Static Data ---

const DRESS_PRODUCTS = [
  // ADDED 'stock' field to simulate inventory management
  { id: 1, name: "Sunburst Cotton Maxi", price: 2999.00, gstRate: 0.12, category: "Cotton Dresses", description: "Light and airy cotton for perfect summer style. Features a breathable fabric and adjustable straps.", imagePlaceholder: "Floral Cotton", colors: ['Pink', 'Yellow', 'Blue'], sizes: ['S', 'M', 'L', 'XL'], reviews: 4.5, stock: 15 },
  { id: 2, name: "Ivory A-Line Cocktail", price: 3450.00, gstRate: 0.18, category: "Party Wear", description: "Elegant A-line dress with a subtle shimmer, perfect for evening events. Knee-length with detailed neckline.", imagePlaceholder: "A-Line", colors: ['Ivory', 'Black', 'Red'], sizes: ['XS', 'S', 'M', 'L'], reviews: 4.7, stock: 0 }, // Out of Stock Example
  { id: 3, name: "Royal Velvet Gown", price: 5999.00, gstRate: 0.28, category: "Party Wear", description: "Luxurious velvet gown for grand evenings. Deep V-neck and a flowing train. Dry clean only.", imagePlaceholder: "Gown", colors: ['Wine', 'Emerald', 'Black'], sizes: ['S', 'M'], reviews: 4.9, stock: 5 }, // Low Stock Example
  { id: 4, name: "Indigo Shirt Dress", price: 1950.00, gstRate: 0.12, category: "Casual Wear", description: "Effortlessly chic denim shirt dress. Comfortable fit with a tie-up belt for definition.", imagePlaceholder: "Denim", colors: ['Indigo', 'Sky Blue'], sizes: ['M', 'L', 'XL'], reviews: 4.2, stock: 25 },
  { id: 5, name: "Chikankari Kurta Dress", price: 4200.00, gstRate: 0.05, category: "Ethnic Wear", description: "Traditional Chikankari embroidery on soft cotton. Perfect blend of heritage and modern silhouette.", imagePlaceholder: "Chikankari", colors: ['White', 'Peach'], sizes: ['S', 'M', 'L'], reviews: 4.6, stock: 10 },
  { id: 6, name: "Bubblegum Mesh Dress", price: 3800.00, gstRate: 0.18, category: "New Arrivals", description: "Trendy mesh overlay dress, just added! Lightweight and stylish for a night out.", imagePlaceholder: "Mesh Pink", colors: ['Pink', 'Grey', 'White'], sizes: ['XS', 'S', 'M'], reviews: 4.3, stock: 7 },
];

const CATEGORIES = ["Cotton Dresses", "Party Wear", "Casual Wear", "Ethnic Wear", "New Arrivals"];
const SIZES = ['XS', 'S', 'M', 'L', 'XL'];
const COLORS = ['Pink', 'Yellow', 'Blue', 'Ivory', 'Black', 'Red', 'Wine', 'Emerald', 'Indigo', 'Sky Blue', 'White', 'Peach', 'Grey'];

const STATE_DATA = { "TN": { name: "Tamil Nadu", code: "TN" }, "DL": { name: "Delhi", code: "DL" }, "MH": { name: "Maharashtra", code: "MH" }, "KA": { name: "Karnataka", code: "KA" } };
const SELLER_STATE = STATE_DATA.TN.code; 
const SHIPPING_FEE = 50.00; 

// --- Primary Color Definitions ---
const ACCENT = 'pink-600';
const ACCENT_LIGHT = 'pink-100';
const BG_COLOR = 'amber-50';
const BUTTON_HOVER = 'pink-700';

// --- Core App Component ---

const App = () => {
  const [cart, setCart] = useState([]);
  const [wishlist, setWishlist] = useState([]);
  const [view, setView] = useState('home'); // 'home' | 'shop' | 'product' | 'cart' | 'checkout' | 'about' | 'contact'
  const [selectedProduct, setSelectedProduct] = useState(null);
  const [buyerState, setBuyerState] = useState(STATE_DATA.DL.code);
  const [searchQuery, setSearchQuery] = useState('');
  const [filters, setFilters] = useState({ category: 'All', size: 'All', priceRange: [0, 6000] });

  // --- Cart and Wishlist Management ---

  const handleAddToCart = (product, quantity = 1) => {
    if (product.stock <= 0) {
        document.getElementById('message-box').classList.remove('hidden');
        document.getElementById('message-text').textContent = `${product.name} is currently out of stock.`;
        return;
    }

    setCart(prevCart => {
      const existingItem = prevCart.find(item => item.id === product.id);
      if (existingItem) {
        return prevCart.map(item =>
          item.id === product.id ? { ...item, quantity: item.quantity + quantity } : item
        );
      } else {
        return [...prevCart, { ...product, quantity }];
      }
    });
    // Give feedback, but stay on current page
    document.getElementById('message-box').classList.remove('hidden');
    document.getElementById('message-text').textContent = `${quantity} x ${product.name} added to cart!`;
  };
  
  const handleToggleWishlist = (productId) => {
    setWishlist(prevList => 
      prevList.includes(productId) 
        ? prevList.filter(id => id !== productId) 
        : [...prevList, productId]
    );
  };

  const handleUpdateQuantity = (id, delta) => {
    setCart(prevCart => {
      const newCart = prevCart.map(item =>
        item.id === id ? { ...item, quantity: item.quantity + delta } : item
      ).filter(item => item.quantity > 0);
      return newCart;
    });
  };

  const handleRemoveItem = (id) => {
    setCart(prevCart => prevCart.filter(item => item.id !== id));
  };

  // --- GST and Pricing Calculation ---

  const calculatedSummary = useMemo(() => {
    const subtotal = cart.reduce((acc, item) => acc + (item.price * item.quantity), 0);
    const isIntraState = SELLER_STATE === buyerState;

    let totalTaxAmount = 0;
    let taxDetails = { CGST: 0, SGST: 0, IGST: 0 };

    cart.forEach(item => {
      const itemTaxableValue = item.price * item.quantity;
      const itemTaxAmount = itemTaxableValue * item.gstRate;
      totalTaxAmount += itemTaxAmount;

      if (isIntraState) {
        const halfRateTax = itemTaxAmount / 2;
        taxDetails.CGST += halfRateTax;
        taxDetails.SGST += halfRateTax;
      } else {
        taxDetails.IGST += itemTaxAmount;
      }
    });

    const grandTotal = subtotal + totalTaxAmount + SHIPPING_FEE;

    return {
      subtotal: parseFloat(subtotal.toFixed(2)),
      taxDetails: {
        CGST: parseFloat(taxDetails.CGST.toFixed(2)),
        SGST: parseFloat(taxDetails.SGST.toFixed(2)),
        IGST: parseFloat(taxDetails.IGST.toFixed(2)),
      },
      totalTaxAmount: parseFloat(totalTaxAmount.toFixed(2)),
      grandTotal: parseFloat(grandTotal.toFixed(2)),
      isIntraState,
    };
  }, [cart, buyerState]);

  // --- Filtering and Search Logic ---

  const filteredProducts = useMemo(() => {
    return DRESS_PRODUCTS.filter(product => {
      // 1. Search Filter
      const matchesSearch = product.name.toLowerCase().includes(searchQuery.toLowerCase()) || 
                            product.description.toLowerCase().includes(searchQuery.toLowerCase());
      if (!matchesSearch) return false;

      // 2. Category Filter
      const matchesCategory = filters.category === 'All' || product.category === filters.category;
      if (!matchesCategory) return false;
      
      // 3. Size Filter (Check if product offers the filtered size)
      const matchesSize = filters.size === 'All' || product.sizes.includes(filters.size);
      if (!matchesSize) return false;

      // 4. Price Filter
      const matchesPrice = product.price >= filters.priceRange[0] && product.price <= filters.priceRange[1];
      if (!matchesPrice) return false;

      return true;
    });
  }, [searchQuery, filters]);

  // --- UI Layout Components ---

  const Navbar = () => (
    <header className="bg-white shadow-lg sticky top-0 z-50">
      <div className="max-w-7xl mx-auto p-4 flex justify-between items-center">
        {/* Logo and Tagline */}
        <div 
          className={`text-xl sm:text-2xl font-extrabold text-${ACCENT} cursor-pointer flex items-center transition-transform duration-300 hover:scale-[1.03]`}
          onClick={() => setView('home')}
        >
          <Tag className={`w-5 h-5 sm:w-6 sm:h-6 mr-1 sm:mr-2 text-amber-500`} /> UNIQUE KINGDOM
        </div>
        
        {/* Navigation Links (Hidden on small screens) */}
        <nav className="hidden md:flex space-x-6 text-gray-700 font-semibold">
          {[{ label: 'Home', view: 'home' }, { label: 'Shop', view: 'shop' }, { label: 'About Us', view: 'about' }, { label: 'Contact', view: 'contact' }].map(link => (
            <button 
              key={link.view} 
              onClick={() => setView(link.view)}
              className={`hover:text-${ACCENT} transition relative group`}
            >
              {link.label}
              <span className={`absolute bottom-0 left-0 w-full h-0.5 bg-${ACCENT} transform scale-x-0 group-hover:scale-x-100 transition-transform duration-300`}></span>
            </button>
          ))}
        </nav>

        {/* Action Buttons (Cart & Login) */}
        <div className="flex items-center space-x-3 sm:space-x-4">
          {/* Wishlist Button (Optional) */}
          <button
            className={`relative p-2 rounded-full bg-amber-100 text-amber-600 hover:bg-amber-200 transition-transform duration-300 hover:scale-[1.1]`}
            aria-label="Wishlist"
          >
            <Heart className="w-5 h-5 sm:w-6 sm:h-6" />
            {wishlist.length > 0 && (
              <span className="absolute top-0 right-0 inline-flex items-center justify-center px-1.5 py-0.5 text-xs font-bold leading-none text-white transform translate-x-1/2 -translate-y-1/2 bg-red-600 rounded-full">
                {wishlist.length}
              </span>
            )}
          </button>
          {/* Cart Button */}
          <button
            onClick={() => setView('cart')}
            className={`relative p-2 rounded-full bg-${ACCENT_LIGHT} text-${ACCENT} hover:bg-pink-200 transition-transform duration-300 hover:scale-[1.1]`}
            aria-label={`Cart with ${cart.length} items`}
          >
            <ShoppingCart className="w-5 h-5 sm:w-6 sm:h-6" />
            {cart.length > 0 && (
              <span className="absolute top-0 right-0 inline-flex items-center justify-center px-1.5 py-0.5 text-xs font-bold leading-none text-white transform translate-x-1/2 -translate-y-1/2 bg-red-600 rounded-full">
                {cart.reduce((sum, item) => sum + item.quantity, 0)}
              </span>
            )}
          </button>
        </div>
      </div>
      
      {/* Mobile Navigation (Hidden on large screens) */}
      <nav className={`md:hidden flex justify-around border-t border-${ACCENT_LIGHT} p-2 text-xs font-semibold text-gray-700`}>
        {[{ label: 'Home', view: 'home', icon: Home }, { label: 'Shop', view: 'shop', icon: Globe }, { label: 'About', view: 'about', icon: Info }, { label: 'Contact', view: 'contact', icon: Phone }].map(link => (
          <button 
            key={link.view} 
            onClick={() => setView(link.view)}
            className={`flex flex-col items-center p-1 rounded-lg ${view === link.view ? `text-${ACCENT} bg-${ACCENT_LIGHT}` : 'hover:text-pink-500'}`}
          >
            <link.icon className="w-5 h-5" />
            <span className='mt-0.5'>{link.label}</span>
          </button>
        ))}
      </nav>
    </header>
  );

  const Footer = () => (
    <footer className={`bg-gray-800 text-gray-300 p-8 mt-12`}>
      <div className="max-w-7xl mx-auto grid grid-cols-2 md:grid-cols-4 gap-8">
        {/* 1. Unique Kingdom Info */}
        <div>
          <h4 className="text-lg font-bold text-white mb-4">UNIQUE KINGDOM</h4>
          <p className="text-sm">Every Dress Has a Story.</p>
          <div className="flex space-x-3 mt-4">
            <Globe className="w-5 h-5 hover:text-white transition cursor-pointer" />
            <Tag className="w-5 h-5 hover:text-white transition cursor-pointer" />
            <Phone className="w-5 h-5 hover:text-white transition cursor-pointer" />
          </div>
        </div>
        {/* 2. Quick Links */}
        <div>
          <h4 className="text-lg font-bold text-white mb-4">Shop</h4>
          {CATEGORIES.slice(0, 4).map(cat => (
            <p key={cat} className="text-sm cursor-pointer hover:text-pink-500 transition mb-1" onClick={() => { setFilters({ ...filters, category: cat }); setView('shop'); }}>{cat}</p>
          ))}
        </div>
        {/* 3. Customer Service */}
        <div>
          <h4 className="text-lg font-bold text-white mb-4">Support</h4>
          <p className="text-sm cursor-pointer hover:text-pink-500 transition mb-1" onClick={() => setView('contact')}>Contact Us</p>
          <p className="text-sm cursor-pointer hover:text-pink-500 transition mb-1">Shipping & Returns</p>
          <p className="text-sm cursor-pointer hover:text-pink-500 transition mb-1">FAQ</p>
        </div>
        {/* 4. Contact Details - Updated with CEO Info */}
        <div>
          <h4 className="text-lg font-bold text-white mb-4">Connect</h4>
          <p className="text-sm mb-1">Email: support@uniquekingdom.com</p>
          <p className="text-sm mb-1">Phone: +91 98765 43210</p>
          <p className="text-sm mb-1">Address: Chennai, TN, India</p>
          
          <h4 className="text-lg font-bold text-white mt-4 mb-2">CEO Contact: Abishek R V</h4>
          <p className="text-sm mb-1 flex items-center"><Mail className="w-4 h-4 mr-2"/> abhishekvadivelrv@gmail.com</p>
          <p className="text-sm mb-1 flex items-center"><Phone className="w-4 h-4 mr-2"/> 9840889119</p>
          <p className="text-sm mb-1 flex items-center"><MapPin className="w-4 h-4 mr-2"/> Krishnagiri, TN, India</p>
        </div>
      </div>
      <div className="border-t border-gray-700 mt-8 pt-6 text-center text-sm text-gray-500">
        &copy; {new Date().getFullYear()} Unique Kingdom. All rights reserved.
      </div>
    </footer>
  );

  // --- Page Components ---

  const HomePage = () => (
    <div className="max-w-7xl mx-auto">
      {/* Hero Section */}
      <div 
        className={`relative h-[60vh] sm:h-[80vh] bg-cover bg-center flex flex-col justify-center items-center text-center p-4 rounded-b-3xl shadow-xl`}
        style={{ backgroundImage: `url('https://placehold.co/1200x800/f0abf2/000000?text=Unique+Kingdom')`}}
      >
        <div className={`absolute inset-0 bg-black opacity-40 rounded-b-3xl`}></div>
        <div className="relative z-10 text-white animate-fadeIn">
          <h1 className="text-5xl sm:text-7xl font-extrabold mb-4 drop-shadow-lg" style={{ fontFamily: 'Georgia, serif' }}>
            UNIQUE KINGDOM
          </h1>
          <p className="text-xl sm:text-3xl font-light mb-8 italic drop-shadow-md">
            Every Dress Has a Story
          </p>
          <button
            onClick={() => setView('shop')}
            className={`px-8 py-3 text-lg font-bold bg-amber-500 text-gray-900 rounded-full shadow-2xl transition-all duration-300 hover:bg-amber-400 hover:scale-[1.05]`}
          >
            Shop Now
          </button>
        </div>
      </div>
      
      {/* Featured Categories */}
      <div className="p-4 sm:p-8 mt-8">
        <h2 className={`text-3xl font-bold text-gray-800 mb-6 text-center border-b-4 border-${ACCENT} pb-2 inline-block mx-auto`}>Our Collections</h2>
        <div className="grid grid-cols-2 lg:grid-cols-4 gap-6 mt-6">
          {CATEGORIES.slice(0, 4).map((cat, index) => (
            <div 
              key={cat} 
              onClick={() => { setFilters({ ...filters, category: cat }); setView('shop'); }}
              className={`relative h-48 bg-white rounded-xl shadow-lg overflow-hidden cursor-pointer group transition-all duration-500 hover:shadow-2xl`}
            >
              <img
                src={`https://placehold.co/400x400/fbcfe8/be185d?text=${cat.replace(/\s/g, '+')}`}
                alt={cat}
                className="w-full h-full object-cover group-hover:scale-[1.1] transition-transform duration-500"
              />
              <div className="absolute inset-0 bg-black bg-opacity-30 group-hover:bg-opacity-50 transition-opacity flex items-center justify-center">
                <p className="text-white text-xl font-bold tracking-wider">{cat}</p>
              </div>
            </div>
          ))}
        </div>
      </div>
    </div>
  );

  const ShopPage = () => {
    // Component to display filters
    const FilterSidebar = () => (
      <div className="bg-white p-6 rounded-xl shadow-xl sticky top-24 space-y-6">
        <h3 className={`text-xl font-bold text-${ACCENT} border-b pb-2 mb-4 flex items-center`}>
          <Filter className="w-5 h-5 mr-2" /> Filter Options
        </h3>
        
        {/* Category Filter */}
        <div>
          <label className="font-semibold text-gray-700 mb-2 block">Category</label>
          <select
            value={filters.category}
            onChange={(e) => setFilters({ ...filters, category: e.target.value })}
            className="w-full p-2 border border-gray-300 rounded-lg focus:border-pink-500 focus:ring-pink-500 transition"
          >
            <option value="All">All Categories</option>
            {CATEGORIES.map(cat => <option key={cat} value={cat}>{cat}</option>)}
          </select>
        </div>

        {/* Size Filter */}
        <div>
          <label className="font-semibold text-gray-700 mb-2 block">Size</label>
          <select
            value={filters.size}
            onChange={(e) => setFilters({ ...filters, size: e.target.value })}
            className="w-full p-2 border border-gray-300 rounded-lg focus:border-pink-500 focus:ring-pink-500 transition"
          >
            <option value="All">All Sizes</option>
            {SIZES.map(size => <option key={size} value={size}>{size}</option>)}
          </select>
        </div>

        {/* Price Range Filter (Simplified) */}
        <div>
          <label className="font-semibold text-gray-700 mb-2 block">Max Price</label>
          <input
            type="range"
            min="1000"
            max="6000"
            step="100"
            value={filters.priceRange[1]}
            onChange={(e) => setFilters({ ...filters, priceRange: [0, parseInt(e.target.value)] })}
            className={`w-full h-2 bg-${ACCENT_LIGHT} rounded-lg appearance-none cursor-pointer`}
          />
          <p className="text-center text-sm mt-1">Up to ₹{filters.priceRange[1]}</p>
        </div>
        
        {/* Reset Button */}
        <button
          onClick={() => setFilters({ category: 'All', size: 'All', priceRange: [0, 6000] })}
          className="w-full py-2 bg-gray-200 text-gray-700 font-semibold rounded-lg hover:bg-gray-300 transition"
        >
          Reset Filters
        </button>
      </div>
    );

    // Component for product card
    const ShopProductCard = ({ product }) => {
      const isWishlisted = wishlist.includes(product.id);
      return (
        <div className="bg-white rounded-xl shadow-lg hover:shadow-2xl transition-shadow duration-300 overflow-hidden relative group">
          <button
            onClick={() => handleToggleWishlist(product.id)}
            className={`absolute top-3 right-3 p-2 rounded-full z-10 transition-all duration-300 ${isWishlisted ? 'text-red-500 bg-white' : 'text-gray-400 bg-white/70 hover:text-red-500 hover:bg-white'}`}
            aria-label="Toggle Wishlist"
          >
            <Heart className={`w-5 h-5 fill-current ${isWishlisted ? 'fill-red-500' : 'fill-none'}`} />
          </button>
          
          <div onClick={() => { setSelectedProduct(product); setView('product'); }} className="cursor-pointer">
            <img
              src={`https://placehold.co/400x500/fecaca/9d174d?text=${product.imagePlaceholder.replace(/\s/g, '+')}`}
              alt={product.name}
              className="w-full h-80 object-cover group-hover:scale-[1.05] transition-transform duration-500"
            />
          </div>
          <div className="p-4">
            <h3 className="text-lg font-semibold text-gray-800 truncate">{product.name}</h3>
            <p className="text-sm text-gray-500 mt-1">{product.category}</p>
            <p className={`text-2xl font-bold text-${ACCENT} mt-1 mb-3`}>₹{product.price.toFixed(2)}</p>
            
            {/* NEW FEATURE: Inventory Status */}
            <div className="text-sm font-semibold mb-3">
                {product.stock > 10 ? (
                    <span className="text-green-600 flex items-center"><CheckCircle className="w-4 h-4 mr-1"/> In Stock</span>
                ) : product.stock > 0 ? (
                    <span className="text-orange-500 flex items-center"><Tag className="w-4 h-4 mr-1"/> Low Stock ({product.stock})</span>
                ) : (
                    <span className="text-red-500 flex items-center"><X className="w-4 h-4 mr-1"/> Out of Stock</span>
                )}
            </div>
            
            <button
              onClick={() => handleAddToCart(product)}
              disabled={product.stock === 0} // Disable if out of stock
              className={`mt-2 w-full py-2 text-white font-semibold rounded-lg shadow-md transition-all duration-150 flex items-center justify-center 
                          ${product.stock === 0 ? 'bg-gray-400 opacity-70 cursor-not-allowed' : `bg-${ACCENT} hover:bg-${BUTTON_HOVER} hover:scale-[1.02]`}`}
            >
              {product.stock === 0 ? 'Sold Out' : <><Plus className="w-4 h-4 mr-2" /> Add to Cart</>}
            </button>
          </div>
        </div>
      );
    };

    return (
      <div className="max-w-7xl mx-auto p-4 sm:p-8">
        <h2 className="text-4xl font-bold text-gray-800 mb-8">Shop All Dresses</h2>
        
        {/* Search Bar */}
        <div className="mb-8 flex">
          <div className="relative w-full max-w-lg mx-auto">
            <Search className="absolute left-4 top-1/2 transform -translate-y-1/2 w-5 h-5 text-gray-400" />
            <input
              type="text"
              placeholder="Search by name or description..."
              value={searchQuery}
              onChange={(e) => setSearchQuery(e.target.value)}
              className="w-full p-3 pl-12 border border-gray-300 rounded-full shadow-inner focus:outline-none focus:border-pink-500 focus:ring-1 focus:ring-pink-500 transition"
            />
          </div>
        </div>
        
        {/* Main Content Grid */}
        <div className="grid grid-cols-1 md:grid-cols-4 gap-8">
          {/* Filters (1st Column on Desktop) */}
          <div className="md:col-span-1">
            <FilterSidebar />
          </div>

          {/* Products (3 Columns on Desktop) */}
          <div className="md:col-span-3">
            {filteredProducts.length > 0 ? (
              <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
                {filteredProducts.map(product => (
                  <ShopProductCard key={product.id} product={product} />
                ))}
              </div>
            ) : (
              <div className="text-center p-12 bg-white rounded-xl shadow-lg">
                <p className="text-xl font-semibold text-gray-600">No dresses found matching your criteria.</p>
                <button 
                    onClick={() => { setSearchQuery(''); setFilters({ category: 'All', size: 'All', priceRange: [0, 6000] }); }}
                    className={`mt-4 py-2 px-4 bg-${ACCENT} text-white rounded-lg hover:bg-${BUTTON_HOVER} transition`}
                >
                    Clear Search & Filters
                </button>
              </div>
            )}
          </div>
        </div>
      </div>
    );
  };

  const ProductDetailPage = ({ product }) => {
    const [selectedSize, setSelectedSize] = useState(product.sizes[0]);
    const [selectedColor, setSelectedColor] = useState(product.colors[0]);
    const [quantity, setQuantity] = useState(1);
    
    // Reset state when product changes
    useEffect(() => {
        if (product) {
            setSelectedSize(product.sizes[0]);
            setSelectedColor(product.colors[0]);
            setQuantity(1);
        }
    }, [product]);

    if (!product) return null;

    const mockReviews = [
      { user: "Priya S.", rating: 5, comment: "Absolutely stunning! The fit is perfect and the velvet quality is top-notch." },
      { user: "Rahul K.", rating: 4, comment: "Bought this for my wife, she loves the design. A bit pricey but worth it." },
    ];

    return (
      <div className="max-w-6xl mx-auto p-4 sm:p-8">
        <button onClick={() => setView('shop')} className={`mb-6 flex items-center text-${ACCENT} hover:text-${BUTTON_HOVER} transition`}>
            <ArrowLeft className="w-5 h-5 mr-2" /> Back to Shop
        </button>

        <div className="grid grid-cols-1 lg:grid-cols-2 gap-8 bg-white p-6 rounded-xl shadow-2xl">
          {/* Image Gallery */}
          <div>
            <img
              src={`https://placehold.co/800x1000/fecaca/9d174d?text=${product.imagePlaceholder.replace(/\s/g, '+')}`}
              alt={product.name}
              className="w-full h-auto object-cover rounded-xl shadow-lg"
            />
          </div>

          {/* Product Details */}
          <div>
            <p className="text-sm text-gray-500 mb-2">{product.category}</p>
            <h1 className="text-4xl font-extrabold text-gray-800 mb-3">{product.name}</h1>
            <p className={`text-3xl font-bold text-${ACCENT} mb-4`}>₹{product.price.toFixed(2)}</p>
            
            <div className="flex items-center mb-6">
                <span className="text-amber-500 text-lg mr-2">{'★'.repeat(Math.floor(product.reviews))}</span>
                <span className="text-gray-500 text-sm">({product.reviews} / 5.0)</span>
            </div>

            <p className="text-gray-700 mb-6 leading-relaxed">{product.description}</p>
            
            {/* Options: Size */}
            <div className="mb-6">
                <p className="font-semibold text-gray-700 mb-2">Available Sizes:</p>
                <div className="flex space-x-2">
                    {product.sizes.map(size => (
                        <button
                            key={size}
                            onClick={() => setSelectedSize(size)}
                            className={`px-4 py-2 border rounded-full font-medium transition ${selectedSize === size ? `bg-${ACCENT} text-white shadow-md` : 'bg-gray-100 border-gray-300 hover:bg-gray-200'}`}
                        >
                            {size}
                        </button>
                    ))}
                </div>
            </div>

            {/* Options: Color */}
            <div className="mb-8">
                <p className="font-semibold text-gray-700 mb-2">Available Colors:</p>
                <div className="flex space-x-3">
                    {product.colors.map(color => (
                        <div 
                            key={color} 
                            onClick={() => setSelectedColor(color)}
                            className={`w-8 h-8 rounded-full border-2 cursor-pointer transition ${selectedColor === color ? `border-${ACCENT} p-0.5` : 'border-gray-300'}`}
                            style={{ backgroundColor: color.toLowerCase() === 'ivory' ? '#fcf9e8' : color.toLowerCase() }}
                            title={color}
                        >
                            <div className="w-full h-full rounded-full" style={{ backgroundColor: color.toLowerCase() === 'ivory' ? '#fcf9e8' : color.toLowerCase() }}></div>
                        </div>
                    ))}
                </div>
            </div>

            {/* Quantity and Add to Cart */}
            <div className="flex items-center space-x-4 mb-8">
                <div className="flex items-center border border-gray-300 rounded-lg">
                    <button onClick={() => setQuantity(q => Math.max(1, q - 1))} className={`p-2 hover:bg-${ACCENT_LIGHT} rounded-l-lg`}><Minus className="w-5 h-5" /></button>
                    <span className="p-2 font-semibold w-8 text-center">{quantity}</span>
                    <button onClick={() => setQuantity(q => q + 1)} className={`p-2 hover:bg-${ACCENT_LIGHT} rounded-r-lg`}><Plus className="w-5 h-5" /></button>
                </div>
                <button
                    onClick={() => handleAddToCart(product, quantity)}
                    disabled={product.stock === 0}
                    className={`flex-grow py-3 text-white font-bold rounded-xl shadow-lg transition-all duration-300 flex items-center justify-center 
                                ${product.stock === 0 ? 'bg-gray-400 opacity-70 cursor-not-allowed' : `bg-${ACCENT} hover:bg-${BUTTON_HOVER} hover:scale-[1.01]`}`}
                >
                    {product.stock === 0 ? 'Sold Out' : <><ShoppingCart className="w-5 h-5 mr-2" /> Add to Cart</>}
                </button>
            </div>
            
            {/* Reviews Section */}
            <div className="mt-8 border-t pt-6">
                <h3 className="text-2xl font-bold text-gray-800 mb-4">Customer Reviews</h3>
                <div className="space-y-4">
                    {mockReviews.map((review, index) => (
                        <div key={index} className="p-4 bg-gray-50 rounded-lg border border-gray-200">
                            <div className="flex justify-between items-center">
                                <p className="font-semibold">{review.user}</p>
                                <span className="text-amber-500 text-sm">{'★'.repeat(review.rating)}</span>
                            </div>
                            <p className="text-sm text-gray-600 mt-1 italic">"{review.comment}"</p>
                        </div>
                    ))}
                </div>
            </div>

          </div>
        </div>
      </div>
    );
  };
    const TaxRow = ({ label, value, colorClass = 'text-gray-700' }) => (
    <div className="flex justify-between py-1 text-sm font-medium">
      <span className={colorClass}>{label}</span>
      <span className={colorClass}>₹{value.toFixed(2)}</span>
    </div>
  );

  const CartPage = () => {
    if (cart.length === 0) {
      return (
        <div className="p-8 text-center max-w-xl mx-auto mt-16">
          <ShoppingCart className="w-16 h-16 text-gray-400 mx-auto mb-4" />
          <h2 className="text-2xl font-semibold text-gray-700">Your Shopping Cart is Empty</h2>
          <p className="text-gray-500 mt-2">Time to find the perfect dress!</p>
          <button 
            onClick={() => setView('shop')}
            className={`mt-6 py-2 px-6 bg-${ACCENT} text-white font-bold rounded-lg hover:bg-${BUTTON_HOVER} transition shadow-md`}
          >
            Continue Shopping
          </button>
        </div>
      );
    }

    const CartItem = ({ item }) => (
        <div className="flex flex-col sm:flex-row items-start sm:items-center p-4 bg-white rounded-xl shadow-sm border border-gray-100 mb-4 transition-all duration-300 hover:shadow-md">
            <img
                src={`https://placehold.co/100x120/fecaca/9d174d?text=${item.imagePlaceholder.replace(/\s/g, '+')}`}
                alt={item.name}
                className="w-20 h-24 object-cover rounded-lg mr-4 mb-2 sm:mb-0"
            />
            <div className="flex-grow">
                <p className="font-semibold text-gray-800">{item.name}</p>
                <p className={`text-lg font-bold text-${ACCENT} mt-1`}>₹{(item.price * item.quantity).toFixed(2)}</p>
                <p className="text-xs text-gray-500">@ ₹{item.price.toFixed(2)} each | GST: {(item.gstRate * 100).toFixed(0)}%</p>
            </div>
            <div className="flex items-center space-x-2 mt-2 sm:mt-0">
                <div className="flex items-center border border-gray-300 rounded-lg">
                    <button
                        onClick={() => handleUpdateQuantity(item.id, -1)}
                        className={`p-1 hover:bg-${ACCENT_LIGHT} rounded-l-lg transition`}
                    >
                        <Minus className="w-4 h-4" />
                    </button>
                    <span className="font-medium w-6 text-center">{item.quantity}</span>
                    <button
                        onClick={() => handleUpdateQuantity(item.id, 1)}
                        className={`p-1 hover:bg-${ACCENT_LIGHT} rounded-r-lg transition`}
                    >
                        <Plus className="w-4 h-4" />
                    </button>
                </div>
                <button
                    onClick={() => handleRemoveItem(item.id)}
                    className="p-2 ml-2 text-red-500 hover:text-red-700 transition"
                    aria-label="Remove Item"
                >
                    <X className="w-5 h-5" />
                </button>
            </div>
        </div>
    );

    return (
      <div className="max-w-4xl mx-auto p-4 sm:p-8">
        <div className="flex items-center mb-8">
          <button onClick={() => setView('shop')} className={`text-${ACCENT} hover:text-${BUTTON_HOVER} transition mr-4`}>
            <ArrowLeft className="w-6 h-6" />
          </button>
          <h2 className="text-3xl font-bold text-gray-800">Your Shopping Cart</h2>
        </div>

        <div className="lg:grid lg:grid-cols-3 lg:gap-8">
          {/* Cart Items List - Column 1 & 2 */}
          <div className="lg:col-span-2 space-y-4">
            {cart.map(item => <CartItem key={item.id} item={item} />)}
          </div>

          {/* Order Summary - Column 3 */}
          <div className="lg:col-span-1 mt-6 lg:mt-0 sticky top-24">
            <div className="bg-white p-6 rounded-xl shadow-lg border border-indigo-100">
              <h3 className="text-xl font-semibold mb-4 text-indigo-700">Order Summary</h3>
              
              {/* Buyer State Selection for GST Logic */}
              <div className="mb-4 p-3 bg-indigo-50 rounded-lg">
                <label htmlFor="buyerState" className="flex items-center text-sm font-semibold text-gray-700 mb-1">
                    <MapPin className="w-4 h-4 mr-2" /> Destination State
                </label>
                <select
                  id="buyerState"
                  value={buyerState}
                  onChange={(e) => setBuyerState(e.target.value)}
                  className="w-full p-2 border border-indigo-300 rounded-lg text-sm focus:ring-indigo-500 focus:border-indigo-500"
                >
                  {Object.values(STATE_DATA).map(state => (
                    <option key={state.code} value={state.code}>
                      {state.name} ({state.code})
                    </option>
                  ))}
                </select>
              </div>

              {/* Price Breakdown */}
              <div className="space-y-2 pb-4 border-b border-gray-200">
                <TaxRow label="Total Items (Taxable Value)" value={calculatedSummary.subtotal} />
                
                {/* Dynamic GST Display */}
                <p className="text-xs font-bold text-gray-600 pt-2">
                    {calculatedSummary.isIntraState ? "Intra-State GST (CGST + SGST)" : "Inter-State GST (IGST)"}
                </p>
                {calculatedSummary.taxDetails.CGST > 0 && <TaxRow label="CGST" value={calculatedSummary.taxDetails.CGST} colorClass="text-green-600" />}
                {calculatedSummary.taxDetails.SGST > 0 && <TaxRow label="SGST" value={calculatedSummary.taxDetails.SGST} colorClass="text-green-600" />}
                {calculatedSummary.taxDetails.IGST > 0 && <TaxRow label="IGST" value={calculatedSummary.taxDetails.IGST} colorClass="text-green-600" />}
                
                <TaxRow label="Shipping & Logistics" value={SHIPPING_FEE} />
              </div>

              <div className={`pt-4 flex justify-between items-center font-extrabold text-2xl text-${ACCENT}`}>
                <span>Grand Total</span>
                <span>₹{calculatedSummary.grandTotal.toFixed(2)}</span>
              </div>
              
              <button
                onClick={() => setView('checkout')}
                className={`mt-6 w-full py-3 bg-${ACCENT} text-white font-bold rounded-xl shadow-lg transition duration-150 hover:bg-${BUTTON_HOVER} transform hover:scale-[1.01]`}
              >
                Proceed to Checkout
              </button>
            </div>
          </div>
        </div>
      </div>
    );
  };

  const CheckoutPage = () => {
    const [formData, setFormData] = useState({ name: '', email: '', address: '', state: buyerState });

    const handleChange = (e) => {
      const { name, value } = e.target;
      setFormData(prev => ({ ...prev, [name]: value }));
      if (name === 'state') setBuyerState(value);
    };

    const handleProceedToPayment = (e) => {
        e.preventDefault();
        // Simple form validation
        if (!formData.name || !formData.email || !formData.address) {
            document.getElementById('message-box').classList.remove('hidden');
            document.getElementById('message-text').textContent = "Please fill out all required fields (Name, Email, Address) to proceed.";
            return;
        }

        // PCI DSS SAQ A COMPLIANCE: Payment logic is simulated as an outsourced redirection.
        console.log("Simulating: Redirecting to Hosted Payment Page for SAQ A compliance...");
        document.getElementById('message-box').classList.remove('hidden');
        document.getElementById('message-text').textContent = `Order placed for ₹${calculatedSummary.grandTotal.toFixed(2)}. PCI SAQ A compliance: Payment processing is outsourced and simulated.`;
        
        // Clear cart after successful "payment" simulation
        setCart([]);
    };

    const Input = ({ label, name, type = 'text', required = true }) => (
        <div className="mb-4">
            <label htmlFor={name} className="block text-sm font-medium text-gray-700">{label}{required && <span className='text-red-500'>*</span>}</label>
            <input
                type={type}
                id={name}
                name={name}
                value={formData[name]}
                onChange={handleChange}
                required={required}
                className="mt-1 block w-full p-3 border border-gray-300 rounded-lg shadow-sm focus:border-pink-500 focus:ring-pink-500"
            />
        </div>
    );
    
    return (
        <div className="max-w-4xl mx-auto p-4 sm:p-8">
            <div className="flex items-center mb-8">
                <button onClick={() => setView('cart')} className={`text-${ACCENT} hover:text-${BUTTON_HOVER} transition mr-4`}>
                    <ArrowLeft className="w-6 h-6" />
                </button>
                <h2 className="text-3xl font-bold text-gray-800">Final Checkout</h2>
            </div>

            <form onSubmit={handleProceedToPayment} className="grid grid-cols-1 lg:grid-cols-5 gap-8">
                {/* Shipping & Contact Info (Col 1-3) */}
                <div className="lg:col-span-3 bg-white p-6 rounded-xl shadow-2xl h-fit">
                    <h3 className="text-2xl font-semibold mb-6 text-indigo-700 flex items-center border-b pb-2">
                        <MapPin className="w-6 h-6 mr-2" /> 1. Shipping Details
                    </h3>
                    <Input label="Full Name" name="name" />
                    <Input label="Email Address" name="email" type="email" />
                    <Input label="Shipping Address (Street, City, Zip)" name="address" />
                    
                    <div className="mb-4">
                        <label htmlFor="state" className="block text-sm font-medium text-gray-700">State/Province<span className='text-red-500'>*</span></label>
                        <select
                            id="state"
                            name="state"
                            value={formData.state}
                            onChange={handleChange}
                            required
                            className="mt-1 block w-full p-3 border border-gray-300 rounded-lg shadow-sm focus:border-pink-500 focus:ring-pink-500"
                        >
                            {Object.values(STATE_DATA).map(state => (
                                <option key={state.code} value={state.code}>{state.name} ({state.code})</option>
                            ))}
                        </select>
                    </div>
                </div>

                {/* Order Summary & Payment (Col 4-5) */}
                <div className="lg:col-span-2 sticky top-24">
                    <div className="bg-white p-6 rounded-xl shadow-2xl">
                        <h3 className="text-2xl font-semibold mb-4 text-indigo-700 border-b pb-2 flex items-center">
                            <ShoppingCart className="w-6 h-6 mr-2" /> 2. Order Summary
                        </h3>
                        
                        {/* Summary lines */}
                        <div className="space-y-1">
                            <TaxRow label="Subtotal (Taxable Value)" value={calculatedSummary.subtotal} />
                            <TaxRow label="Total GST" value={calculatedSummary.totalTaxAmount} colorClass="text-green-600"/>
                            <TaxRow label="Shipping" value={SHIPPING_FEE} />
                        </div>
                        
                        <div className="mt-4 pt-4 border-t border-dashed border-gray-300">
                            <div className={`flex justify-between font-extrabold text-3xl text-${ACCENT}`}>
                                <span>TOTAL</span>
                                <span>₹{calculatedSummary.grandTotal.toFixed(2)}</span>
                            </div>
                        </div>

                        {/* Payment Button */}
                        <div className="mt-6 p-4 bg-pink-50 rounded-xl shadow-inner">
                            <div className='flex items-center text-sm font-semibold text-gray-700 mb-3'>
                                <CheckCircle className='w-4 h-4 mr-2 text-green-600' /> Secure Payment via Hosted Gateway
                            </div>
                            <button
                                type="submit"
                                className={`w-full py-3 px-4 bg-${ACCENT} text-white font-bold rounded-xl shadow-lg transition duration-150 hover:bg-${BUTTON_HOVER} transform hover:scale-[1.01] focus:outline-none focus:ring-4 focus:ring-pink-300`}
                            >
                                Pay ₹{calculatedSummary.grandTotal.toFixed(2)} Now
                            </button>
                        </div>
                    </div>
                </div>
            </form>
        </div>
    );
  };
  
  const AboutUsPage = () => (
    <div className="max-w-4xl mx-auto p-4 sm:p-8">
      <h2 className="text-4xl font-bold text-gray-800 mb-6 text-center">Our Story: Unique Kingdom</h2>
      <div className="bg-white p-6 sm:p-10 rounded-xl shadow-2xl">
        
        <img
            src={`https://placehold.co/800x400/fff7ed/d97706?text=Our+Mission`}
            alt="Unique Kingdom Mission"
            className="w-full h-auto object-cover rounded-lg mb-6 shadow-lg"
        />

        <p className={`text-xl font-semibold mb-6 text-${ACCENT}`}>
            "Every Dress Has a Story." This is the philosophy that guides us.
        </p>

        <h3 className="text-2xl font-bold text-gray-800 mt-8 mb-4">Our Mission</h3>
        <p className="text-gray-700 mb-6 leading-relaxed">
            Unique Kingdom was founded on the belief that fashion should not compromise comfort. 
            Our mission is simple: to deliver **stylish, comfortable, and high-quality cotton dresses** to women everywhere. 
            We meticulously source the finest natural fabrics, ensuring every stitch tells a story of ethical production and timeless elegance. 
        </p>
        
        <h3 className="text-2xl font-bold text-gray-800 mt-8 mb-4">The Cotton Difference</h3>
        <p className="text-gray-700 mb-6 leading-relaxed">
            While we offer a range of party and ethnic wear, our heart lies in our cotton collection. 
            We believe cotton is the king of comfort. It's breathable, durable, and naturally beautiful. 
            We design pieces that transition effortlessly from a busy workday to a relaxed evening, making sustainable style accessible.
        </p>

        <h3 className="text-2xl font-bold text-gray-800 mt-8 mb-4">Our Values</h3>
        <ul className="list-disc list-inside text-gray-700 space-y-2 ml-4">
            <li>**Quality First:** We stand by the enduring quality of our fabrics.</li>
            <li>**Ethical Sourcing:** Ensuring fair wages and sustainable practices in our supply chain.</li>
            <li>**Timeless Design:** Creating pieces that stay relevant beyond fast fashion trends.</li>
        </ul>
      </div>
    </div>
  );

  const ContactPage = () => (
    <div className="max-w-4xl mx-auto p-4 sm:p-8">
      <h2 className="text-4xl font-bold text-gray-800 mb-6 text-center">Get In Touch</h2>
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
        
        {/* Contact Form */}
        <div className="bg-white p-6 rounded-xl shadow-2xl">
            <h3 className="text-2xl font-semibold mb-6 text-indigo-700 border-b pb-2">Send Us a Message</h3>
            <form onSubmit={(e) => { e.preventDefault(); document.getElementById('message-box').classList.remove('hidden'); document.getElementById('message-text').textContent = "Thank you! Your message has been sent and we will respond within 48 hours."; }}>
                {/* Reusing the Input component, but need state for Contact form */}
                <div className="mb-4">
                    <label htmlFor="contactName" className="block text-sm font-medium text-gray-700">Your Name<span className='text-red-500'>*</span></label>
                    <input
                        type="text"
                        id="contactName"
                        name="contactName"
                        required={true}
                        className="mt-1 block w-full p-3 border border-gray-300 rounded-lg shadow-sm focus:border-pink-500 focus:ring-pink-500"
                    />
                </div>
                <div className="mb-4">
                    <label htmlFor="contactEmail" className="block text-sm font-medium text-gray-700">Your Email<span className='text-red-500'>*</span></label>
                    <input
                        type="email"
                        id="contactEmail"
                        name="contactEmail"
                        required={true}
                        className="mt-1 block w-full p-3 border border-gray-300 rounded-lg shadow-sm focus:border-pink-500 focus:ring-pink-500"
                    />
                </div>
                <div className="mb-4">
                    <label htmlFor="message" className="block text-sm font-medium text-gray-700">Message</label>
                    <textarea 
                        id="message" 
                        name="message" 
                        rows="4" 
                        required 
                        className="mt-1 block w-full p-3 border border-gray-300 rounded-lg shadow-sm focus:border-pink-500 focus:ring-pink-500"
                    ></textarea>
                </div>
                <button
                    type="submit"
                    className={`w-full py-3 bg-${ACCENT} text-white font-bold rounded-xl shadow-lg transition-all duration-150 hover:bg-${BUTTON_HOVER} hover:scale-[1.01]`}
                >
                    Submit Inquiry
                </button>
            </form>
        </div>

        {/* Contact Info - Updated with CEO Info */}
        <div className="bg-white p-6 rounded-xl shadow-2xl space-y-6">
            <h3 className="text-2xl font-semibold mb-6 text-indigo-700 border-b pb-2">Our Support Details</h3>
            
            <div className="flex items-start space-x-3">
                <Mail className={`w-6 h-6 text-${ACCENT}`} />
                <div>
                    <p className="font-semibold text-gray-800">Email Support</p>
                    <p className="text-gray-600">support@uniquekingdom.com</p>
                </div>
            </div>

            <div className="flex items-start space-x-3">
                <Phone className={`w-6 h-6 text-${ACCENT}`} />
                <div>
                    <p className="font-semibold text-gray-800">Phone</p>
                    <p className="text-gray-600">+91 98765 43210 (Mon-Fri, 10am - 6pm IST)</p>
                </div>
            </div>

            <div className="flex items-start space-x-3">
                <MapPin className={`w-6 h-6 text-${ACCENT}`} />
                <div>
                    <p className="font-semibold text-gray-800">Head Office</p>
                    <p className="text-gray-600">Unique Kingdom Fashions, Chennai, Tamil Nadu, India</p>
                </div>
            </div>
            
            {/* CEO Details Section */}
            <h3 className="text-2xl font-semibold mb-6 text-indigo-700 border-b pb-2 pt-4">CEO Contact: Abishek R V</h3>
        
            <div className="flex items-start space-x-3">
                <Mail className={`w-6 h-6 text-${ACCENT}`} />
                <div>
                    <p className="font-semibold text-gray-800">CEO Email</p>
                    <p className="text-gray-600">abhishekvadivelrv@gmail.com</p>
                </div>
            </div>

            <div className="flex items-start space-x-3">
                <Phone className={`w-6 h-6 text-${ACCENT}`} />
                <div>
                    <p className="font-semibold text-gray-800">CEO Contact Number</p>
                    <p className="text-gray-600">9840889119</p>
                </div>
            </div>
            
            <div className="flex items-start space-x-3">
                <MapPin className={`w-6 h-6 text-${ACCENT}`} />
                <div>
                    <p className="font-semibold text-gray-800">CEO Location</p>
                    <p className="text-gray-600">Krishnagiri, Tamil Nadu, India</p>
                </div>
            </div>

            
            <div className="pt-4 border-t border-gray-200">
                <p className="font-semibold text-gray-800 mb-3">Follow Us</p>
                <div className="flex space-x-4">
                    <Globe className={`w-6 h-6 text-gray-600 hover:text-${ACCENT} transition cursor-pointer`} />
                    <Tag className={`w-6 h-6 text-gray-600 hover:text-${ACCENT} transition cursor-pointer`} />
                    <Heart className={`w-6 h-6 text-gray-600 hover:text-${ACCENT} transition cursor-pointer`} />
                </div>
            </div>
        </div>
      </div>
    </div>
  );


  const renderView = () => {
    switch (view) {
      case 'shop':
        return <ShopPage />;
      case 'product':
        return <ProductDetailPage product={selectedProduct} />;
      case 'cart':
        return <CartPage />;
      case 'checkout':
        return <CheckoutPage />;
      case 'about':
        return <AboutUsPage />;
      case 'contact':
        return <ContactPage />;
      case 'home':
      default:
        return <HomePage />;
    }
  };
  
  const handleCloseMessage = () => {
    document.getElementById('message-box').classList.add('hidden');
  };

  return (
    // Applied font-inter class and cream background
    <div className={`min-h-screen bg-${BG_COLOR} font-inter`}>
      <script src="https://cdn.tailwindcss.com"></script>
      <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');
        /* Ensures smooth transitions for hover effects */
        .transition-all {'{'} transition: all 0.3s ease-in-out; {'}'} 
        
        /* Message Box Animation */
        @keyframes bounceIn {'{'}
            0% {'{'} transform: scale(0.9); opacity: 0; {'}'}
            100% {'{'} transform: scale(1); opacity: 1; {'}'}
        {'}'}
        .animate-bounceIn {'{'}
            animation: bounceIn 0.3s ease-out;
        {'}'}
      </style>
      
      <Navbar />
      <main className="min-h-[70vh]">
        {renderView()}
      </main>
      <Footer />
      
      {/* Custom Message Box (Replaces alert()) */}
      <div id="message-box" className="fixed inset-0 bg-gray-900 bg-opacity-50 flex items-center justify-center hidden z-[100]">
        <div className={`bg-white p-6 rounded-xl shadow-2xl max-w-sm w-full animate-bounceIn`}>
          <h3 className={`text-xl font-bold mb-3 text-${ACCENT}`}>Notification</h3>
          <p id="message-text" className="text-gray-700 mb-4"></p>
          <button
            onClick={handleCloseMessage}
            className={`w-full py-2 bg-${ACCENT} text-white font-semibold rounded-lg hover:bg-${BUTTON_HOVER}`}
          >
            Got It!
          </button>
        </div>
      </div>
    </div>
  );
};

export default App;
